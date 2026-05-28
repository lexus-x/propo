# Project 3 — Reasoning-Trace Causality in VLAs

> ⚠️ **SCOOP ALERT (partial — read before proceeding).** *Altered Thoughts, Altered Actions* (arXiv **2603.12717**, Mar 2026) already executes the **content-ablation rung** on a reasoning VLA over 40 LIBERO tasks, including the **length-matched filler-token control**. It finds the same asymmetry we predicted: swapping object names hurts (up to −45 pp on individual tasks), while sentence reordering, spatial-direction reversal, token noise, and a 70B-LLM's "plausible-but-wrong" plans are all within ±4 pp. **Rung 1 is largely pre-empted, and the compute-control (rung 3) is partly pre-empted.** It is framed as an *adversarial-robustness* paper, not a causal attribution study, and does **not** run activation patching or any train-time/test-time decomposition. **Pivot:** the surviving novelty is (a) **rung 2 — activation patching** to show whether action heads *read* the trace, (b) **rung 4 — train-time vs test-time decomposition**, and (c) a **unified 3-way attribution** (content / test-time budget / train-time regularization). We re-scope the project around these and treat content-ablation as a *replication-and-extension* baseline that cites 2603.12717 prominently.

## TL;DR (3-4 lines)
Reasoning VLAs report gains from chain-of-thought (CoT), but nobody has cleanly attributed *why*. We hypothesize the gain comes mostly from **extra test-time compute** and **train-time auxiliary supervision/regularization**, not from the **semantic content** of the generated plan. A 4-rung causal ladder (content ablation, **activation patching**, compute control, **train/test decomposition**) on public ECoT/CoT-VLA checkpoints decomposes success into content-vs-budget-vs-supervision. Outcome is a principled steer for the field; recent work (2603.12717) already supports the "content is largely inert" side, which we extend mechanistically.

## The gap / motivation
Every recent reasoning VLA — ECoT (2407.08693), CoT-VLA (2503.22020), ACoT-VLA (2601.11404), InstructVLA (2507.17520) — adds an intermediate reasoning step and reports higher success, then attributes the gain to "reasoning." This is a **correlation**: CoT changes the token budget, the training objective, *and* the semantic plan simultaneously. In pure LLMs we already know these confounds dissociate — filler tokens can substitute for CoT on some tasks (*Let's Think Dot by Dot*, 2404.15758). No VLA paper has run the controls that separate them. If the plan's *content* is inert, the field is paying a heavy interpretability/latency tax for what is really a compute knob or a training regularizer — and should redirect effort to latent test-time-compute scaling instead.

## Falsifiable hypothesis (+ refuting outcome)
**H:** CoT improves VLA success **primarily via (i) extra forward compute and (ii) train-time auxiliary supervision**, **not via the semantic content** of the generated plan.
- **Prediction:** corrupting/shuffling reasoning content while preserving token budget leaves success largely intact; removing the budget hurts; a model *trained* with CoT but *evaluated* without it retains most of the gain over a model never trained with CoT.
- **Refuting outcome:** content-corruption (esp. beyond entity swaps) collapses success while length-matched filler does **not**, **and** activation-patching a correct trace's hidden states into a corrupted rollout restores success → the trace is genuinely causal/semantic and the field should keep investing in it.
- **Calibration note:** 2603.12717 already pushes evidence toward H *except* that **entity references are causal**. Our sharpened H therefore predicts a **narrow** semantic channel (object grounding) carried by content, with planning/sequencing being budget+supervision. This is itself testable and falsifiable.

## Precise novelty & positioning

| Paper | arXiv id | What they claim | What they never causally tested |
|---|---|---|---|
| ECoT | 2407.08693 | Embodied CoT (plan/subtask/grounding) before action raises success | Whether trace *content* (vs budget/supervision) does the work |
| CoT-VLA | 2503.22020 | Visual subgoal CoT beats SOTA by 6–17% | Compute-matched / content-corrupted / patched controls |
| ACoT-VLA | 2601.11404 | Explicit+implicit action reasoning improves policy | Content-vs-compute attribution; train/test decomposition |
| InstructVLA | 2507.17520 | Instruction tuning + embodied reasoning lifts manipulation | Is reasoning causal at inference or a training regularizer? |
| Mech-interp steering VLA | 2509.00328 | Linear feature directions (speed/dir) causally steer actions | Patches *behavioral* features, **not reasoning-token→action-head reading** |
| Observing/Controlling Features | 2603.05487 | Features are linearly observable/controllable in VLAs | Does not target CoT trace tokens or compute/supervision attribution |
| **Altered Thoughts (scoop)** | **2603.12717** | Corrupting CoT: only entity refs matter; order/noise/wrong-plan inert | **No activation patching; no train/test decomposition; security framing, not attribution** |
| Let's Think Dot by Dot | 2404.15758 | Filler tokens can replace CoT in LLMs (compute, not content) | Not a VLA; no embodied action head |

**Net gap (post-pivot):** no work isolates whether action heads **read** the reasoning trace (mechanistic mediation), nor decomposes the gain into **content / test-time budget / train-time supervision** within a single controlled VLA study.

## Experimental design
**Models (public, reasoning checkpoints):** ECoT (OpenVLA-based, weights + code public) as primary; CoT-VLA as a visual-CoT replication; OpenVLA / π₀ as non-reasoning controls (both used in 2509.00328). **Benchmarks:** LIBERO (in-dist), SimplerEnv (Google-Robot + WidowX/Bridge, visual-matching & variant-aggregation), **LIBERO-Plus (2510.13626)** for OOD perturbation. **Metric:** success-rate **attribution** — Δ-success decomposed into content, budget, and train-supervision components (with CIs over ≥3 seeds × task suites).

**Rung 1 — Content ablation** *(replication + extension of 2603.12717).* Conditions: correct / shuffled / wrong-but-fluent / generic length-matched filler / none. Extension beyond the scoop: (a) separate **entity-only** vs **plan-structure-only** corruptions to localize the narrow semantic channel; (b) run on **SimplerEnv + LIBERO-Plus OOD**, not just in-dist LIBERO.

**Rung 2 — Activation patching (the headline contribution).** Patch reasoning-token hidden states from a *correct* rollout into a *corrupted* one (and vice-versa), layer- and position-resolved, measuring success recovery/destruction. This is a **causal-mediation** test of whether action heads actually consume the trace — never done for reasoning tokens in a VLA. Requires open weights + forward-hook access (ECoT/OpenVLA/π₀ qualify).

**Rung 3 — Compute control.** Length-matched dummy/"pause" tokens (à la 2404.15758) vs meaningful plan, holding token count fixed, to separate *more forward compute* from *meaningful plan*. (Filler partly covered by 2603.12717; we add a monotonic budget sweep to estimate the compute–success slope.)

**Rung 4 — Train-time vs test-time (the second contribution).** Compare: (a) **trained-with-CoT / evaluated-with**, (b) **trained-with-CoT / evaluated-without** (drop the think segment at inference), (c) **trained-without-CoT**. Gap (a→b) = test-time-content value; gap (b→c) = pure train-time regularization value. Light fine-tunes only; no pretraining from scratch.

## Expected contribution
A **principled, mechanistically-grounded answer** to: *should the field invest in generating reasoning traces, or is the benefit test-time compute + a training regularizer?* Concretely, a decomposition `Δ_success ≈ Δ_content + Δ_budget + Δ_supervision` with confidence intervals across in-dist and OOD sim. If content's share is small (as 2603.12717 hints), we provide the causal/patching evidence that justifies redirecting effort toward **latent test-time-compute scaling** and **CoT-as-training-regularizer** rather than expensive, latency-heavy explicit traces.

## Why it's publishable + target venue + acceptance hook
**Venue:** CoRL (embodied + interpretability fit) or ICLR/NeurIPS (mechanistic-interp track). **Hook:** challenges a *fashionable* assumption ("reasoning helps because the model reasons") with **causal-mediation rigor** (activation patching) that the closest scoop did **not** run, plus a clean train/test decomposition. **Hard to scoop further:** activation-patching reasoning tokens in VLAs + the 3-way attribution is, as of May 2026, unpublished; 2603.12717 makes the *behavioral* result less novel but **strengthens our motivation** and gives a citable launchpad.

## Honest win-win scorecard
- **This is a STUDY, not a method.** No new VLA is proposed; the deliverable is causal understanding. State this plainly to reviewers.
- **Win (H supported):** field-steering negative-ish result with mechanism → high-impact, but negative results face venue bias; lean on the patching novelty.
- **Win (H refuted):** traces are genuinely causal → validates the reasoning-VLA program and gives the first mechanistic *why*. Either way the paper resolves a real ambiguity.
- **Pre-empted slice:** rung 1 / filler control overlaps 2603.12717 — disclosed up front; we cite and extend rather than claim.

## Risks & kill-criteria
- **Patching is inconclusive** (distributed, non-localizable trace coding) → report as a finding but the headline weakens. *Kill rung 2 as headline if no layer/position shows >X pp causal effect across seeds; fall back to rung 4.*
- **Checkpoints don't expose internals cleanly** (fused diffusion action heads, e.g. π₀) complicate patching → restrict rung 2 to autoregressive ECoT/OpenVLA where hooks are clean.
- **Confound leakage:** filler tokens shift attention statistics → control with matched token-type distributions and report attention diagnostics.
- **Full scoop appears** (a paper doing patching *and* train/test decomposition) → re-pivot to the OOD-attribution angle (LIBERO-Plus) and entity-channel localization, which remain open.
- **Hard kill:** if a pre-registered replication of 2603.12717 fails to reproduce the entity-vs-structure asymmetry on our checkpoints, pause and re-examine before building rungs 2–4 on it.

## First de-risking experiment (week 1)
On **one ECoT checkpoint** on **LIBERO** (all suites, ≥3 seeds): run rung 1 — correct vs shuffled vs wrong-but-fluent vs **length-matched filler** vs none. **Decision rule:** if filler ≈ correct (within CI) and content-corruption barely moves success, H is supported and we proceed to patching (rung 2) as the headline. If content-corruption collapses success while filler holds, H is refuted early → pivot to *characterizing* the causal semantic channel (likely entity grounding, per 2603.12717). Either branch yields a paper.

## Feasibility
**Sim-only**, **public checkpoints** (ECoT/OpenVLA weights + code public; CoT-VLA, π₀ available), **modest compute** (inference-heavy evals + light fine-tunes for rung 4; no large pretraining). **Activation patching needs model-internals access** — satisfied by ECoT/OpenVLA (HF weights, transformer hooks) and 2509.00328's demonstrated π₀/OpenVLA hooking; **fused diffusion-head VLAs are harder to patch**, so rung 2 is scoped to autoregressive backbones. LIBERO, SimplerEnv, and LIBERO-Plus are all open benchmarks.

## Key references
- Zawalski et al., *Robotic Control via Embodied Chain-of-Thought Reasoning* (ECoT), CoRL 2024 — arXiv:2407.08693
- Zhao et al., *CoT-VLA: Visual Chain-of-Thought Reasoning for VLAs*, CVPR 2025 — arXiv:2503.22020
- *ACoT-VLA: Action Chain-of-Thought for VLAs*, CVPR 2026 — arXiv:2601.11404
- *InstructVLA: VLA Instruction Tuning from Understanding to Manipulation*, 2025 — arXiv:2507.17520
- Haon et al., *Mechanistic Interpretability for Steering VLAs*, CoRL 2025 (PMLR v305) — arXiv:2509.00328
- *Observing and Controlling Features in VLAs*, 2026 — arXiv:2603.05487
- *Altered Thoughts, Altered Actions: Probing CoT Vulnerabilities in VLA Manipulation*, 2026 — arXiv:2603.12717 **(closest prior work / partial scoop)**
- *DeepThinkVLA: Enhancing Reasoning Capability of VLAs*, 2025 — arXiv:2511.15669
- *LIBERO-Plus: In-depth Robustness Analysis of VLAs*, 2025 — arXiv:2510.13626
- Pfau, Merrill, Bowman, *Let's Think Dot by Dot: Hidden Computation in Transformer LMs*, 2024 — arXiv:2404.15758


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
