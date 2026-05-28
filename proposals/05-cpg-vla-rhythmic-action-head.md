# Project 5 — CPG-VLA: Central Pattern Generator Action Head for Rhythmic Manipulation

---

## TL;DR

We replace the action-chunk decoder of a pretrained VLA (OpenVLA-OFT or π0) with a compact Central Pattern Generator (CPG) parameter head. The backbone runs at ~5–10 Hz and outputs a small vector of oscillator parameters (amplitude, frequency, phase, inter-DOF coupling, spatial target); a lightweight differentiable CPG (~50 lines) unrolls those parameters into a 200–500 Hz motor trajectory. The result is faster inference, lower compute, stronger periodic-task sample efficiency, and automatic contact-driven phase re-synchronisation — advantages that action-chunking and diffusion decoders structurally cannot replicate for rhythmic manipulation tasks (wiping, polishing, scrubbing, screw-driving, stirring, sawing).

---

## The Gap / Motivation

Action-chunking (OpenVLA-OFT) and diffusion decoding (π0) share the same representational assumption: an action is a finite, open-loop burst of joint waypoints. This is appropriate for pick-and-place, but actively harmful for rhythmic-contact tasks:

1. **Periodicity is unrepresented.** A chunk of length H encodes one period. When the task requires N > H/T strokes, the policy must either re-issue chunks with perfect phase stitching (it does not) or use an enormous H (wastes capacity, reintroduces latency).

2. **Contact breaks phase.** Wiping a surface with variable friction, or driving a screw through wood grain, produces contact transients that accelerate or retard the end-effector. A chunk has no internal oscillator state to absorb this; it drifts and must wait for the next VLA query. A CPG with sensory feedback coupling (the biological design principle) re-synchronises automatically within a fraction of a cycle.

3. **Compute redundancy.** Running a diffusion decoder at 100 Hz (π0) to produce what is fundamentally a sinusoidal wrist motion is a ~100× compute waste. Diffusion provides generative power that zero-entropy periodic signals do not need.

4. **Sample inefficiency.** Learning the correct wrist frequency from scratch via regression requires many demonstrations. A CPG encodes the periodic inductive bias structurally; the VLA only needs to learn *which* parameters to set, not *how* to be periodic.

No published work (as of 2026-05-28, after targeted search) combines a CPG oscillator as the action head of a full end-to-end VLA backbone for manipulation. The gap is real.

---

## Precise Novelty & Positioning

| Paper | arXiv ID | What they do | Our delta |
|---|---|---|---|
| **Language Movement Primitives** | 2602.02839 | VLM outputs DMP parameters → finite open-loop trajectory | DMP is a one-shot trajectory generator, not a sustained oscillator; no VLA backbone (vision + language + end-to-end training); no contact-driven phase reset |
| **FODMP** | 2603.24806 | Distills ProDMP diffusion into single-step ProDMP parameter prediction | No VLA backbone; uses probabilistic DMPs (finite basis), not self-sustaining oscillators; no contact perturbation handling |
| **AVP** | 2605.22183 | VLM infers visual-primitive tokens → conditions flow-matching action expert | Visual primitives are spatial goals, not oscillator parameters; still uses chunk-level action decoder; not designed for rhythmic tasks |
| **CPG-RL** | 2211.00458 | RL modulates CPG amplitude/frequency for quadruped locomotion | Locomotion only; no language or vision; no manipulation; no transformer VLA backbone |
| **Learning Periodic Tasks** | 2109.14078 | Rhythmic rDMPs from video demos (wiping, stirring) via Bayesian opt | No language; not a VLA; rDMPs are bounded primitives requiring re-triggering; no contact phase feedback |
| **OpenVLA-OFT** | 2502.19645 | Fine-tuning recipe for VLAs with parallel decoding + chunking + L1 loss | Our backbone base; we replace its chunk decoder head |
| **π0** | 2410.24164 | Flow-matching VLA with broad manipulation capability | Our second backbone base; we replace its diffusion action decoder |

**Precise claim:** A CPG oscillator as the VLA action head for robot manipulation, with two-rate control (slow VLA / fast oscillator) and language-driven online parameter resetting, is novel as of the search date.

---

## Method

### 1. CPG Parameter Head

A small MLP head (2–3 layers, ≤512 hidden units) attached to the final VLA token produces a parameter vector **p** per DOF at query rate f_VLA (~5–10 Hz):

```
p = [A_i, ω_i, φ_i (i=1..D), κ_ij (coupling), x_target]
```

- **A_i**: per-DOF amplitude (positive scalar, softplus-activated)  
- **ω_i**: angular frequency (positive scalar, sigmoid × ω_max)  
- **φ_i**: phase offset (unconstrained, interpreted mod 2π)  
- **κ_ij**: D×D sparse coupling matrix (only kinematically linked pairs; sparse attention or fixed topology)  
- **x_target**: 3-D spatial endpoint from vision (wipe centroid, screw axis, etc.)

### 2. The Oscillator (Hopf, ~50 lines)

We use coupled Hopf oscillators (smooth, analytically tractable, differentiable):

```
ẋᵢ = (μ - r²) xᵢ - ωᵢ yᵢ + Σⱼ κᵢⱼ xⱼ
ẏᵢ = (μ - r²) yᵢ + ωᵢ xᵢ + Σⱼ κᵢⱼ yⱼ
```

where rᵢ² = xᵢ² + yᵢ², μ = Aᵢ² controls limit-cycle radius. The output motor command for DOF i is qᵢ = Aᵢ xᵢ + x_target_i. Sensory feedback (joint torque or F/T sensor) perturbs φ via a phase-response curve — the standard biological coupling mechanism. The ODE is integrated at 200–500 Hz with a fixed-step RK4 (trivially fast; no GPU required).

The Matsuoka oscillator is the backup if Hopf proves harder to tune for multi-DOF coupling; van der Pol adds asymmetry (useful for screw-driving approach vs. withdraw). The choice is an ablation.

### 3. Two-Rate Interface

```
[Camera, Language] → VLA backbone (7B, GPU, ~100 ms) → p
                                                          ↓
                               p held constant → Hopf ODE integrator (CPU, ~0.3 ms/step) → q(t)
```

When the VLA re-queries (each ~100–200 ms), the current oscillator phase φ(t) is preserved and the new parameters are applied as a smooth parameter update (linear interpolation over 1–2 cycles to avoid discontinuities). Language instructions reset relevant parameters online: "faster" → increases ω, "wider" → increases A, "stop" → decays A to zero.

### 4. Training

- **Backbone**: Initialise from OpenVLA-OFT (2502.19645) pretrained weights; the chunk-decoder head is discarded and replaced with the CPG parameter head (random init).
- **Supervision signal**: Demonstrations are processed to extract per-DOF **p*** via fitting a Hopf oscillator to the demonstration trajectory (closed-form for A, ω; phase via Hilbert transform). L1 loss on **p** (following OFT's regression objective).
- **Contact robustness fine-tuning**: Add a second phase — short RL episodes where random surface-height perturbations (±3 mm, ±5 deg inclination) force the phase-feedback coupling to be exercised. Reward = task completion (number of strokes, torque achieved).
- **Compute**: Training the head only (backbone frozen, or with LoRA on final 4 layers) requires a single A100 or equivalent. Full fine-tune requires one 80 GB GPU for ~2–3 days.

---

## Experimental Design

### Target Tasks (build on ManiSkill2/3 + real Franka)

| Task | Why rhythmic/contact | Sim environment |
|---|---|---|
| Surface wiping | Variable friction transients; periodic strokes | ManiSkill2 `Wipe-v0` (exists) or custom |
| Polishing / sanding | Sustained force + oscillatory wrist | Custom (ManiSkill3 force-control) |
| Screw-driving | Rotation + axial advance; phase locks to thread | Custom |
| Stirring liquid | Closed-loop orbit; splashing = contact perturbation | Custom (particle fluid) |
| Sawing (foam block) | Rhythmic force + stroke reversal | Custom |

ManiSkill2 provides GPU-parallelised sim and existing demonstrations. At least wiping and stirring can be assembled quickly; others require ~1 week of environment scripting.

### Baselines

1. **OpenVLA-OFT** (2502.19645) — chunk decoding, same backbone
2. **π0** (2410.24164) — diffusion decoding, same backbone
3. **LMP** (2602.02839) — DMP params from VLM (no full VLA)
4. **Ablation: CPG head + no phase feedback** — removes the contact re-sync mechanism
5. **Ablation: Hopf vs. Matsuoka vs. van der Pol** — oscillator model choice

### Metrics

| Metric | Detail |
|---|---|
| Task success rate | Binary completion (n=100 episodes) |
| Demos-to-80% success | Sample efficiency: number of demonstrations required to reach 80% SR |
| Inference control Hz | Effective motor command rate at deployment |
| Phase error under perturbation | Surface raised ±3 mm mid-task; measure cycle-count offset before re-sync |
| GPU FLOP / action step | Compute cost normalised per motor command |

### Ablations

- A1: Phase feedback on vs. off (contact re-sync)
- A2: Oscillator model: Hopf / Matsuoka / van der Pol
- A3: VLA query rate: 2 / 5 / 10 / 25 Hz
- A4: CPG head on non-rhythmic task (pick-and-place) — expected to *match* chunking, not exceed; verifies scope is honest

---

## Why It's Publishable + Target Venue + Acceptance Hook

**Target venues:** CoRL 2026 (primary), RA-L with IROS option (secondary).

**Acceptance hook:** The rhythmic-manipulation blind spot in VLA evaluation. Every major VLA benchmark (LIBERO, OXE, BridgeV2) is dominated by pick-and-place and articulated object tasks. Wiping, polishing, and screw-driving are near-absent from VLA leaderboards despite being among the highest-value industrial and household manipulation tasks. A paper that (a) names this blind spot, (b) demonstrates that existing VLA decoders structurally fail at it, and (c) provides a principled fix with clear neuroscience motivation (CPG as biological precedent) has a clean narrative. CoRL 2026 has historically rewarded neuro-inspired robotics that demonstrates concrete wins on real hardware.

**Minimum publishable result:** CPG-VLA reaches ≥80% success on wiping with ≤50% of the demonstrations required by OpenVLA-OFT chunking, and recovers phase within 1.5 cycles after a ±3 mm surface perturbation vs. no recovery for chunking. Real-robot demo on at least one task is expected by CoRL.

---

## Honest Win-Win Scorecard

| Claim | Verdict | Qualification |
|---|---|---|
| **Novel** | ✓ | CPG as VLA action head for manipulation is unoccupied as of 2026-05-28 |
| **Better (rhythmic tasks)** | ✓ (likely) | Contact re-sync is a structural advantage; sample efficiency gain is expected but must be measured |
| **Better (general tasks)** | ✗ | No advantage over chunking/diffusion for non-periodic tasks; pick-and-place performance will be neutral at best |
| **Less compute** | ✓ | Oscillator integration is trivial CPU load; eliminates diffusion denoising iterations |
| **Faster (motor Hz)** | ✓ | 200–500 Hz output from CPU vs. 25–100 Hz chunk replay; VLA query rate drops to 5–10 Hz |
| **Sample efficient** | ? | Periodic inductive bias should help; unproven until data ablation is run |

The method is a win-win *within its scope* (rhythmic-contact tasks). Outside that scope, it is neutral-to-worse and the paper must say so.

---

## Risks & Kill Criteria

1. **Niche scope.** If reviewers insist that rhythmic tasks are insufficiently important, the paper is rejected regardless of results. Mitigation: include an industrial motivation section (sanding, buffing, fastening = large automation market); one real-robot polishing demo strengthens the case significantly.

2. **Non-rhythmic generalisation.** CPG-VLA cannot handle tasks without a dominant frequency. Language instructions like "pick up the cup" produce no natural oscillator parameters. Mitigation: the system must fall back gracefully (zero amplitude = static target from x_target). This fallback must be demonstrated to not *hurt* relative to baseline on pick-and-place (A4 ablation).

3. **Phase-fitting supervision fails.** If demonstration trajectories are noisy or multi-modal, Hilbert-transform fitting of oscillator parameters may be ill-conditioned. Kill criterion: if fitting error for >30% of demonstrations exceeds 15% of stroke amplitude, the supervision signal is not viable and the method must switch to end-to-end differentiable oscillator unrolling with task-level reward only — feasible but significantly more expensive.

4. **Contact feedback latency.** Real F/T sensor latency (1–2 ms) may be enough for 200 Hz control; USB-connected sensors at 500 Hz may introduce aliasing. Kill criterion: if phase re-sync in hardware requires >3 cycles (vs. ≤1.5 in sim), the sim-to-real gap for the key differentiator is too large.

5. **Oscillator instability.** Multi-DOF Hopf coupling can exhibit bifurcations. If coupling matrix κ requires heavy hand-tuning per task, the approach is not general. Mitigation: learn κ as part of the CPG head; constrain to negative semi-definite (stable by construction).

---

## First De-Risking Experiment (Week 1)

**Goal:** Confirm that Hopf oscillator parameters can be reliably extracted from real demonstration data, and that a small MLP can predict them from language + image embeddings.

**Protocol:**
1. Collect 20 kinesthetic demonstrations of surface wiping on a Franka (or use an existing wiping dataset from BridgeV2/OXE if available).
2. Fit Hopf parameters (A, ω, φ per DOF) to each demo trajectory using Hilbert transform. Compute fit RMSE.
3. Train a small MLP (3 layers, 256 units) on frozen CLIP embeddings (image + language) → CPG parameters. Evaluate held-out prediction error.
4. Unroll predicted parameters in the Hopf ODE and compare resulting trajectory against ground truth via DTW distance.

**Pass criterion:** RMSE of oscillator fit < 10% of stroke amplitude on ≥85% of demonstrations; MLP prediction DTW within 2× of demonstration variance. Both can be done on a laptop CPU in under 3 days, before any GPU training.

If this fails, the oscillator-fitting supervision signal is not viable and the project pivots to end-to-end RL fine-tuning of the CPG head, which is slower but still tractable.

---

## Feasibility for an Academic Lab

- **Hardware:** Single 80 GB GPU (A100 or H100) for VLA fine-tuning; all oscillator work runs on CPU. Real-robot experiments require one 7-DOF arm (Franka / UR5 / xArm); F/T sensor at wrist (~$1K) is needed for contact phase feedback.
- **Timeline:** Week 1 — de-risking experiment above. Weeks 2–3 — CPG head training on sim wiping task (ManiSkill). Week 4 — full sim benchmark (5 tasks, baselines). Week 5–6 — real-robot transfer + perturbation experiments.
- **Code base:** Build on OpenVLA-OFT's public codebase (MIT licence); CPG oscillator is ~50 lines of PyTorch. Total new code volume is small.
- **Risk-adjusted scope:** A single-GPU lab can produce a credible CoRL submission with sim results on 3 tasks + real-robot demo on 1 task within ~6 weeks. Full 5-task benchmark is a stretch goal for the camera-ready.

---

## Key References

| Title | Venue / Year | arXiv ID |
|---|---|---|
| Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success (OpenVLA-OFT) | RSS 2025 | 2502.19645 |
| π0: A Vision-Language-Action Flow Model for General Robot Control | RSS 2025 | 2410.24164 |
| Language Movement Primitives: Grounding Language Models in Robot Motion | arXiv 2026 | 2602.02839 |
| FODMP: Fast One-Step Diffusion of Movement Primitives Generation | arXiv 2026 | 2603.24806 |
| Action with Visual Primitives (AVP) | arXiv 2026 | 2605.22183 |
| CPG-RL: Learning Central Pattern Generators for Quadruped Locomotion | RA-L 2022 | 2211.00458 |
| Learning Periodic Tasks from Human Demonstrations | ICRA 2022 | 2109.14078 |

---

*Proposal written 2026-05-28. Scoop search: 8 targeted queries across arXiv, Semantic Scholar, and open web; no CPG-as-VLA-action-head manipulation paper found.*


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
