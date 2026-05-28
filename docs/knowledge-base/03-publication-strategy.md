# Publication Strategy for VLA Work

[← Knowledge Base](README.md) · [← Repo README](../../README.md)

Where strong VLA work actually gets published, what reviewers reward vs. reject, and which paper *types* give an academic lab the best acceptance-odds-per-unit-effort.

## Bottom line
VLA is **conference-dominated, not journal-dominated.** The canonical papers (OpenVLA, π₀, RT-2, OpenVLA-OFT) landed at **CoRL / RSS / NeurIPS / ICLR** — not Science Robotics or T-RO. Journals are too slow for foundation-model pace. For a modestly-resourced academic lab the realistic high-value targets are **CoRL, RA-L, and a NeurIPS/ICLR-class venue (incl. their Datasets & Benchmarks tracks)**, and the best effort-to-acceptance ratio comes from **analysis/empirical-study and benchmark papers run in simulation**, not new large models needing robot fleets.

## Venue comparison
| Venue | Type | VLA receptiveness | Real-robot expectation | Novelty vs rigor | ~Accept rate | Timeline |
|---|---|---|---|---|---|---|
| **CoRL** | Conf | **Very high** (de-facto home) | Encouraged, **not required**; sim OK w/ transfer evidence | Balanced; **Limitations section mandatory** | ~39–40% | ~5–6 mo, 1 rebuttal |
| **RSS** | Conf | Very high, prestigious | Strongly valued; sim tolerated w/ justification | **High novelty bar**; multi-round R&R | ~20–30% | long, 2-stage |
| **RA-L** | Journal | **High & growing** | Valued but **sim acceptable**; rigor>scale | Rigor & clarity; 6–8 pg | **36–38%** | **fast: ≤6 mo, ~4 avg** |
| **NeurIPS / ICLR** | ML conf | High for method/efficiency/analysis | **Not required** | ML framing; method novelty OR strong empirical insight | ~25% | ~5–6 mo |
| **ICRA / IROS** | Conf | High volume, lower prestige/paper | Valued; sim widely accepted | Lower novelty bar, breadth | ~43–46% | ~4–5 mo |
| **CVPR / ICCV** | CV conf | Moderate–high (perception/robustness framing) | Not required | Visual-perception novelty | ~22–27% | ~5 mo |
| **AAAI** | Conf | Moderate–high (efficiency/analysis) | Not required | Method/efficiency novelty | ~23% | ~4 mo |
| **Science Robotics** | Journal | Low frequency, very high bar | Essentially **required** + major impact | Landmark | very low | 6–12+ mo |
| **T-RO / IJRR / Nature MI** | Journal | Low for VLA (too slow vs field pace) | Strongly expected | Depth/completeness/theory | selective | 9–18 mo |

*Caveat:* public acceptance-rate repos lag; ICRA/IROS rates are historical norms; treat CoRL as ~35–40%.

## What gets accepted vs rejected (top-3 realistic targets)
**CoRL (best primary target).** Accepted: clear robot-learning contribution; honest mandatory **Limitations**; recognized-benchmark eval (LIBERO/SimplerEnv/CALVIN) with **multiple seeds + baselines**; real-robot *or* explicit sim-to-real evidence; code/video. Rejected: no robotics focus (desk-reject); single-seed/cherry-picked; missing Limitations; "+2% with no insight"; "generalist" claim from one setup.

**RA-L (best journal path; fastest).** Accepted: tight, rigorous, well-written 6–8 pg single idea; clean ablations; sim-only fine. Rejected: over-scoped; sloppy experiments; dual-submission; reviewers feel it belongs in a full journal.

**NeurIPS/ICLR (+ Datasets & Benchmarks track).** Accepted: strong empirical insight OR a genuinely useful benchmark/dataset; ML framing (hypotheses, controlled variables); multi-model evaluation. Rejected: systems framing w/o ML novelty; trivial benchmark reskin; insufficient model coverage; leaderboard with no analysis.

**Cross-cutting rejection triggers:** memorization masquerading as generalization (test compositional/perturbed splits); no baselines vs OpenVLA/π₀/Octo; single seed; overclaiming "generalist"; missing failure analysis.

## Recent accepted VLA papers and their "acceptance hook"
| Paper | Venue | Hook |
|---|---|---|
| **OpenVLA** | CoRL 2024 | Open model + dataset + SOTA → became the reference baseline |
| **OpenVLA-OFT** | 2025 | Efficiency + SOTA recipe (97.1% LIBERO, 26×) — a *recipe*, not a new model |
| **MoLe-VLA** | AAAI 2025 | Efficiency (dynamic layer activation, 36.8% faster) |
| **ReconVLA** | AAAI 2025 | Analysis→method (VLAs mis-allocate visual attention; fix it) |
| **VLAS** | ICLR 2025 | New modality (speech) + 2 datasets |
| **DreamVLA** | NeurIPS 2025 | Method novelty (world-model integration) |
| **"What Can RL Bring to VLA Generalization?"** | NeurIPS 2025 | **Pure empirical study** — PPO vs GRPO vs DPO; *no new model* ([2505.19789](https://arxiv.org/abs/2505.19789)) |
| **LIBERO-Plus / -PRO** | benchmark venues | Robustness benchmark + critique ("memorization not generalization") |
| **TinyVLA / SmolVLA** | RA-L 2025 / report | Efficiency / small-model for affordable robotics |

**Pattern:** the winning non-mega-lab hooks are **efficiency, analysis/empirical study, benchmark/robustness, new-modality** — not "bigger model, more robots."

## Paper types by acceptance-odds-per-effort (academic lab)
Assumed constraints: limited/no real robots, moderate GPUs, ability to LoRA-fine-tune ~7B open models (OpenVLA fine-tunes with LoRA on a single A100-40GB; ~4–6 GPU-hr/10k steps on H100).

1. **Analysis / empirical study (BEST ratio).** No new model, no robot, modest compute (LoRA + sim). Hard to dismiss if rigorous. Proven: *What Can RL Bring* → NeurIPS'25. → **Proposals #2, #3.**
2. **Benchmark / robustness eval.** Pure-sim, high reuse, fits NeurIPS/ICLR D&B track. Risk: must be genuinely useful, not a reskin (and this corner is now crowded — see research log).
3. **Efficiency / compression.** Single clear metric (speed/memory/params); strong RA-L/AAAI/ICML fit; sim-evaluable. → **Proposals #6, #8.**
4. **Method (data-efficient adaptation / fine-tuning recipe).** Moderate effort if scoped to *adaptation* of an existing base. Higher novelty bar; needs strong baselines. → **Proposals #1, #5, #7.**
5. **New large-scale model/dataset from real robots (WORST ratio for this profile).** Robot fleets + massive data + compute. Avoid as a first paper.

## Concrete recommendation
Primary play: an **analysis/empirical-study or efficiency/method paper** fine-tuning OpenVLA(-OFT) (and ideally π₀/Octo for breadth) via **LoRA in simulation (LIBERO + SimplerEnv)**, with a sharp hook and an honest Limitations section. Target **CoRL** (deadline-permitting) or **NeurIPS/ICLR (D&B)**; use **RA-L** as the fast fallback for a tighter single-idea version.
