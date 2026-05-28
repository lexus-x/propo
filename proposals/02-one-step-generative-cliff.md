# Project 2 — The One-Step Generative Cliff

## TL;DR (3-4 lines)
One-step (1-NFE) flow / MeanFlow VLAs claim parity with multi-step inference, but every parity claim is measured on saturated LIBERO (~98%) or on a handful of real-robot tasks. We test, on a **single fixed checkpoint**, whether the 1-step-vs-multi-step gap stays flat or **widens monotonically with task difficulty and OOD shift** — peaking on contact-rich, precise, and multimodal-action tasks. Either outcome publishes: a "generative cliff" (gap grows → one-step is unsafe off the saturated regime) or a calibrated "when-is-one-step-safe" map (gap flat → vindication). Sim-only, public checkpoints, inference-dominated compute.

## The gap / motivation
The field is converging on one-step action generation, and the headline evidence looks settled. **SnapFlow** (2604.05656) reports 1-NFE at **98.75%** on LIBERO vs a 10-step teacher at 97.75% — but LIBERO is saturated, and its own abstract notes naive step reduction is "unreliable," dropping 97.75%→96.75%. **OpenVLA-OFT** (2502.19645) pushed LIBERO from 76.5%→97.1%, confirming the ceiling. So when a one-step method "matches" the teacher there, it is matching a number with almost no headroom to lose. The crucial counter-signal is **MeanFlow-VLA** (2603.01469): its *own* NFE ablation shows a brutal cliff — **1→49%, 2→48%, 3→51.5%, 4→79%, 5→84.5%** — yet this is measured **only on three real SO-101 tasks** with no controlled difficulty axis and no sim/OOD sweep. The two data points (saturated-flat vs real-robot-cliff) are never reconciled on a controlled distribution. **No paper sweeps NFE on a fixed checkpoint across a difficulty ladder.** That reconciliation is the contribution.

## Falsifiable hypothesis (+ the null result and why it's still publishable)
**H1 (cliff):** On a fixed checkpoint, success(NFE) is non-decreasing in NFE, and the **1-step-vs-best-NFE gap grows monotonically with task difficulty / OOD shift**, with the steepest slope on tasks requiring precise grasps, contact-rich interaction, long horizons, or genuinely multimodal action distributions. Falsified if the gap is flat (≤2–3 pp) across the entire ladder.
**H0 / null (safe):** The gap stays flat across difficulty. This is **equally publishable** — it converts a community assumption into a *measured* result and yields a calibrated "one-step is safe when {entropy < τ, contact-free, horizon < H}" deployment map. A credible flat result on hard OOD tasks is itself a strong, citable claim (currently nobody has shown it).

## Precise novelty & positioning
| Paper | arXiv id | What they report | What they NEVER tested |
|---|---|---|---|
| SnapFlow | 2604.05656 | 1-NFE = 98.75% on (saturated) LIBERO; real robot | NFE sweep vs *difficulty/OOD*; step-sweep is over exec horizon, not generative NFE |
| MeanFlow-VLA | 2603.01469 | NFE cliff 1→49%…5→84.5% on 3 real tasks | Any sim benchmark; any *controlled* difficulty/OOD axis; resolution of the cliff |
| One-Step Flow Policy | 2603.12480 | 1-step beats 100-step on 56 sim tasks | Per-task gap vs graded difficulty; OOD shift; mechanistic multimodality metrics |
| FlowPolicy | 2412.04987 | Consistency-FM, fast 1-step 3D policy | Fixed-checkpoint NFE×difficulty curve on VLA-scale benchmarks |
| MP1 | 2507.10543 | 1-NFE MeanFlow, 6.8 ms inference | NFE×difficulty slope; mode-collapse diagnostics |
| Fair Benchmarking (one- vs multi-step) | 2603.14186 | Mismatched-NFE comparisons mislead — on **ImageNet** (FID/CLIP) | Anything on robot *action* distributions or VLA task difficulty |
| LIBERO-Plus | 2510.13626 | OOD robustness, L1–L5 difficulty, 95%→<30% drops | Effect of generative **NFE** on that robustness curve |
| π0 / OpenVLA-OFT | 2410.24164 / 2502.19645 | Flow-matching VLA backbones; LIBERO ceilings | Whether their multi-step margin is load-bearing off-saturation |

**Novelty:** first **controlled, same-checkpoint NFE × difficulty-ladder** characterization for VLAs, with mechanistic (distribution-collapse) explanation. 2603.14186 validates our premise but on images, not actions — it is support, not scoop. Concurrent fixes (Dispersive MeanFlow 2601.20701, "From Flow to One Step" 2603.09415, ProbeFlow 2603.17850, DiG-Flow 2512.01715) *assume* a cliff and patch it; none **characterize where/why** it opens.

## Experimental design
- **Fixed checkpoints:** distill one 1-NFE head from a public flow VLA (π0 / OpenVLA-OFT / SnapFlow recipe); **the same weights** are evaluated at **NFE ∈ {1, 2, 4, 5, 10}** (10 = effective teacher). No retraining per NFE — the gap must come from inference only.
- **Difficulty ladder (monotone shift):** LIBERO → LIBERO-Long → SimplerEnv-Fractal → SimplerEnv-Bridge → RoboCasa → **LIBERO-Plus OOD (L1–L5 perturbations: viewpoint, lighting, layout, sensor noise)**.
- **Headline metric:** the **gap-vs-difficulty slope** — regress (success@10 − success@1) against an ordinal difficulty index; report slope, CI, and per-rung gaps with ≥500 rollouts/cell, 3 seeds.
- **Mechanistic metrics:** (1) action-distribution **multimodality / entropy** (sample N actions per state, fit GMM, measure mode count + per-mode mass — does 1-NFE collapse to the mean?); (2) **trajectory jerk / smoothness**; (3) **per-phase failure attribution** (approach / grasp / place) to localize where 1-step breaks.

## Expected result & the mechanistic explanation
We expect H1 on contact-rich/precise rungs: 1-NFE regresses toward the **conditional mean** of a multimodal action posterior, which is harmless when the demonstration distribution is near-unimodal/easy (LIBERO) but catastrophic at narrow grasps and bimodal approach directions, where the mean lands *between* valid modes. Multi-step integration resolves to a single mode. Predicted signature: **GMM mode-mass collapse and entropy drop under 1-NFE correlate with the per-rung gap** (r expected > 0.6), and grasp-phase failures dominate. This converts "one-step is risky" from folklore into a measured, mechanistically-grounded curve.

## Why it's publishable + target venue + acceptance hook
**Venue:** CoRL (primary) or NeurIPS/ICLR Datasets & Benchmarks. **Hook:** a *characterization that ages well* — as benchmarks de-saturate, "where does one-step break and why" only grows more relevant, and the result is hard to scoop because the value is in the controlled grid + mechanistic correlation, not a single SOTA number. Reviewer-proofing: a clean null is still a contribution (calibrated safety map), so the paper is not hostage to a positive result.

## Honest win-win scorecard
**This is a STUDY, not a win-win method — stated plainly.** It proposes no faster/cheaper/stronger policy; it characterizes an existing trade-off.
- Novel: ✓ (no controlled NFE×difficulty VLA study exists)
- Understanding value: **high** (mechanism + deployment map)
- Faster / cheaper / higher-SOTA: ✗ (not the goal)
- Reproducibility / longevity: high (public checkpoints + benchmarks; grid is rerunnable)

## Risks & kill-criteria
- **Saturation everywhere:** if even RoboCasa/LIBERO-Plus-OOD stay flat (≤3 pp), H1 dies — but we *publish the null* as the safety map. (Not a project-killer.)
- **Scoop creep:** the one-step VLA space is hot (≥6 fixes in 2026). **Kill/pivot** if a paper publishes the *same controlled NFE×difficulty grid* on a fixed checkpoint; we then pivot to the mechanistic mode-collapse diagnostic alone.
- **Confound — distillation artifact:** a bad 1-NFE head fakes a cliff. **Mitigate** by also sweeping the *teacher's* native ODE steps (no distillation) so the gap is intrinsic, not distillation damage.
- **Sim≠real:** mechanistic claim validated only in sim; flagged as a limitation, with one optional real-robot spot-check.

## First de-risking experiment (week 1)
Take **one** public 1-NFE checkpoint (SnapFlow-style on π0 / OpenVLA-OFT). Run the NFE sweep {1,2,4,5,10} on **SimplerEnv-Bridge** (a known harder, less-saturated suite) and **LIBERO-Plus OOD (L4–L5)** only. Single question: **does any gap > 3 pp appear off-saturation?** If yes on either → H1 is alive, scale the full ladder. If flat on both → pivot immediately to the calibrated-safety-map framing.

## Feasibility
Sim-only, all public checkpoints + benchmarks (LIBERO, SimplerEnv, RoboCasa, LIBERO-Plus). Compute is **distill-once + inference-only sweep**: ~12 h on 1 GPU to distill a 1-NFE head (SnapFlow reports ~12 h/1 GPU), then the entire NFE×difficulty grid is forward-pass rollouts. No large-scale training. One person, ~4–6 weeks to a full draft.

## Key references
- SnapFlow: One-Step Action Generation for Flow-Matching VLAs via Progressive Self-Distillation — arXiv 2604.05656 (2026)
- Mean-Flow based One-Step Vision-Language-Action — arXiv 2603.01469 (2026)
- One-Step Flow Policy: Self-Distillation for Fast Visuomotor Policies — arXiv 2603.12480 (2026)
- FlowPolicy: Fast and Robust 3D Flow-based Policy via Consistency Flow Matching — arXiv 2412.04987 (AAAI 2025)
- MP1: MeanFlow Tames Policy Learning in 1-step for Robotic Manipulation — arXiv 2507.10543 (2025)
- Fair Benchmarking of Emerging One-Step Generative Models Against Multistep Diffusion and Flow Models — arXiv 2603.14186 (2026)
- LIBERO-Plus: In-depth Robustness Analysis of Vision-Language-Action Models — arXiv 2510.13626 (2025)
- π0: A Vision-Language-Action Flow Model for General Robot Control — arXiv 2410.24164 (2024)
- OpenVLA-OFT (Fine-Tuning VLA Models: Optimizing Speed and Success) — arXiv 2502.19645 (2025)
- Dispersive MeanFlow Policy Optimization ("One Step Is Enough") — arXiv 2601.20701 (2026)
- From Flow to One Step: Multi-Modal Trajectory Policies via IMLE Distribution Distillation — arXiv 2603.09415 (2026)
- ProbeFlow: Training-Free Adaptive Flow Matching for VLA Models — arXiv 2603.17850 (2026)


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
