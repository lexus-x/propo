# Red-Team Analyses — The Detailed Kills

[← Knowledge Base](README.md) · [← Repo README](../../README.md)

The full adversarial reasoning that killed the four leading *method* ideas, plus the validation-round logic. Each was reviewed by a fresh-context agent whose **default verdict was reject**, instructed to find the fatal flaw and hunt for scooping papers. Preserved here because the *reasoning* (specific refutations, salvage angles) is as valuable as the verdict.

---

## Kill 1 — EBT-VLA (energy-based action head)
**Verdict: KILL** (salvageable only as a narrowed multimodality study).
- **Scooped wholesale by EBT-Policy ([2510.27545](https://arxiv.org/abs/2510.27545)):** it already claims ~2-step inference vs diffusion's ~100, *emergent zero-shot recovery from failed grasps with pure BC*, and energy-as-OOD-signal. It even explicitly omits the VLA-scale experiment "to avoid obscuring comparison" — so the proposal = "do the experiment they declined to do" = a scaling exercise.
- **The "free uncertainty" claim is refuted.** An EBT energy is *unnormalized* — no calibration guarantee. *On OOD Detection with Energy-Based Models* ([2107.08785](https://arxiv.org/abs/2107.08785)) shows EBMs don't inherently beat other density estimators at OOD. Meanwhile failure detection is already done *with* calibration (SAFE, conformal) and the test-time-verification niche is owned by *Scaling Verification > Scaling Policy Learning* ([2602.12281](https://arxiv.org/abs/2602.12281), Finn/Pavone) using a *separate* verifier.
- **"Less compute / stable training" contradicts the implicit-BC literature.** Implicit BC ([2109.00137](https://arxiv.org/abs/2109.00137)) is notoriously unstable (InfoNCE negatives); the objective is *biased even at the population level* ([2309.05803](https://arxiv.org/abs/2309.05803)). The "50× fewer iterations" number is from a ~100M from-scratch model, not a 7B backbone. And OpenVLA-OFT's plain L1 head already hits 97.1% LIBERO at 26× throughput → no headroom to demonstrate the speed win.
- **Salvage:** drop speed/recovery/uncertainty; study only whether an energy head's *multi-basin landscape* represents genuinely multimodal action distributions where unimodal L1 collapses to the mean — and front-load a stability-at-7B-scale experiment.

## Kill 2 — MoD-VLA (token-level Mixture-of-Depths)
**Verdict: KILL** (salvageable only as a narrow iso-compute "route-around vs prune" study, RA-L-tier).
- **Pre-scooped to the decimal.** VLA-Pruner ([2511.16449](https://arxiv.org/abs/2511.16449)) at 50% token retention: OpenVLA 102% rel-acc @1.33×, OpenVLA-OFT 101% @1.46× — *the proposal's exact pitch, same backbones, same operating point.* LightVLA ([2509.12594](https://arxiv.org/abs/2509.12594), ICRA'26) owns the "vision-token sparsity" analysis (−59% FLOPs, +2.9% success).
- **The "novel analysis" is a founding premise of a whole subfield** (VLA-Cache 2502.02175 "static visual tokens"; ADP shows redundancy is *phase-dependent*, refuting MoD's fixed 50%).
- **Win-win is dishonest.** Generic importance routing destroys spatial fidelity manipulation needs — FastV on OpenVLA: 52.1→38.1% (−14 pts). The methods that *do* win add action-aware/task-aware machinery + fine-tuning (undercutting "less compute").
- **Salvage:** an iso-FLOP study of whether *route-around* (skipped tokens kept in residual/KV) preserves the spatial fidelity that *hard pruning* destroys on precision tasks.

## Kill 3 — Calibration-free Equivariant VLA
**Verdict: KILL** (salvageable only as an empirical "when does approximate equivariance pay off vs uncalibrated-3D" study).
- **= Eq.Bot ⊕ E2VLA.** Canonical Policy ([2505.18474](https://arxiv.org/abs/2505.18474)) already does learned-canonicalization→SE(3)-equivariant policy; Eq.Bot ([2511.15194](https://arxiv.org/abs/2511.15194)) is already a canonicalization wrapper on OpenVLA-OFT; "canonicalization makes any pretrained model equivariant" is published meta-method ([2310.01647](https://arxiv.org/abs/2310.01647), [2405.14089](https://arxiv.org/abs/2405.14089)).
- **The motivating gap is being closed *without* equivariance.** E2VLA itself names VGGT as the fix; AnyCamVLA ([2603.05868](https://arxiv.org/abs/2603.05868)) is already a calibration-free VLA (no equivariance tax); the VGGT cluster (AugVLA-3D 2602.10698, VGGT-DP 2509.18778) delivers uncalibrated 3D features + 2D-data compatibility.
- **Self-defeating tension.** Canonicalization rotates inputs OOD for a web-scale VLM backbone (documented in 2310.01647); *approximate* equivariance forfeits the exact-symmetry guarantee that *creates* the sample-efficiency — plausibly yielding neither.
- **Salvage:** measure whether learned canonicalization *retains* E2VLA's low-data + 94% cross-embodiment advantage, benchmarked against AnyCamVLA/VGGT-DP (not against the already-conceded E2VLA).

## Kill 4 — Flow-Map / Align-Your-Flow action head
**Verdict: KILL — borderline dead-on-arrival.**
- **The differentiator is moot.** The selling point was "flow maps don't degrade with steps → cures the single-step cliff." But SnapFlow ([2604.05656](https://arxiv.org/abs/2604.05656)) already hits **1-NFE 98.75% LIBERO, *above* its 10-step teacher**, and shows graceful cross-horizon behavior — the cliff it claims to cure doesn't exist on the target benchmark.
- **Zero method novelty.** Decoupled-MeanFlow ([2510.24474](https://arxiv.org/abs/2510.24474)) already established "convert a flow model into a flow map → 1–4 step, cheaper than from-scratch"; Align-Your-Flow ([2506.14603](https://arxiv.org/abs/2506.14603)) already "generalizes consistency + flow-matching, effective across all step counts." The proposal = applying a published image-gen objective to an action head — the same move MeanFlow-VLA/MP1/FlowPolicy/SnapFlow already made.
- **Neither "better" nor "cheaper" survives:** LIBERO saturated (±0.7% noise at 400 eps); the flow-map/JVP objective is *more* expensive per step than SnapFlow's 2-step Euler shortcut targets; both are 1-NFE (same speed).
- **Only sliver:** SnapFlow/MeanFlow-VLA skip SimplerEnv — a difficulty-stratified study *might* find a cliff there (this became Proposal #2, reframed as a *study* not a method).

---

## Validation-round reasoning
- **Eval/robustness benchmark → KILL.** The most-scooped corner: LIBERO-CF ([2602.17659](https://arxiv.org/abs/2602.17659)), LangGap ([2603.00592](https://arxiv.org/abs/2603.00592)), LIBERO-Plus ([2510.13626](https://arxiv.org/abs/2510.13626)), INT-ACT ([2506.09930](https://arxiv.org/abs/2506.09930)), vla-eval ([2603.13966](https://arxiv.org/abs/2603.13966)) — three counterfactual-language benchmarks in ~8 weeks. Only sliver: a *joint factorial* perception×language×execution attribution validated against real-robot ground truth (incremental, high desk-reject-fatigue risk).
- **Win-win method hunt → 1 survivor.** The ≥200 Hz analytic inner-loop, reframed as a *control-bandwidth frontier* result (no VLA reports that axis) — became **Proposal #1**. Multi-rate fast-slow itself is saturated (TIDAL/FAVLA/DuoCore-FS).
- **Mechanistic study hunt → 2 survivors.** *One-step cliff* (NFE×difficulty, the cleanest, lowest-scoop-risk → **Proposal #2**) and *reasoning-trace causality* (causal-intervention ladder → **Proposal #3**, partial scoop by *Altered Thoughts, Altered Actions* 2603.12717, which did content-ablation but not activation patching / train-vs-test decomposition).

## Meta-lesson
A "novel + better + cheaper + faster method that shatters benchmarks" is the **most-contested target in robot learning** — 4/4 of the strongest method ideas died to Feb–Apr 2026 preprints. Durable contributions came from (a) **cross-disciplinary imports** into the saturated stack and (b) **characterization studies** that sell understanding rather than racing a technique.
