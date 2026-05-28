# Open Problems & Evaluation Landscape (2023–2026)

[← Knowledge Base](README.md) · [← Repo README](../../README.md)

The documented weaknesses of current VLAs, ranked by how open they remain, plus the benchmark catalog and its known flaws. *Severity* = impact × prevalence; *Openness* = how unsolved as of mid-2026; *Active* = whether many groups are attacking it now (high activity = a clean single-paper contribution is harder to land).

## Ranked open problems (most → least open)

**1. Evaluation & reproducibility crisis (most open; foundational).** Real-robot eval is unscalable/unsafe/non-reproducible; sim has a reality gap; cross-lab setups aren't standardized → fair comparison is impossible. Named the top bottleneck in *10 Open Challenges* ([2511.05936](https://arxiv.org/abs/2511.05936)) and proven by a 2025–26 wave of responses (RoboArena [2506.18123](https://arxiv.org/abs/2506.18123), AutoEval [2503.24278](https://arxiv.org/abs/2503.24278), RobotArena∞ [2510.23571](https://arxiv.org/abs/2510.23571)). *No standard has won yet.* Severity: critical. Active: intensely — but fragmented.

**2. Robustness brittleness / shortcut learning.** The most damning recent finding: SOTA VLAs drop ~95% → <30% under modest camera-viewpoint or init-state perturbation, yet are *insensitive to language* — they exploit vision→action shortcuts and often ignore the instruction. Reported success rates overstate competence. Cites: LIBERO-Plus ([2510.13626](https://arxiv.org/abs/2510.13626)), LangGap ([2603.00592](https://arxiv.org/abs/2603.00592)), When-Vision-Overrides-Language / LIBERO-CF ([2602.17659](https://arxiv.org/abs/2602.17659)). Severity: critical. Active: emerging — diagnosis recent, mitigation open.

**3. Data scarcity & collection cost.** Teleop ~$1k–10k/hr, ~1 human-hr per robot-hr; largest teleop sets ~100,000× smaller than LLM corpora. Drives world-model/video/sim data generation. Cites: [2511.05936](https://arxiv.org/abs/2511.05936) (Challenge #3). Severity: critical. Active: very (synthetic/world-model) — crowded; data-*quality*/curation angles remain open (→ Proposal #4).

**4. Weak 3D / spatial reasoning.** VLAs inherit 2D encoders; depth/geometry reasoning is weak → poor precise placement / clutter. Diagnosis settled; right architecture not. Cites: FALCON ([2510.17439](https://arxiv.org/abs/2510.17439)), DepthVLA ([2510.13375](https://arxiv.org/abs/2510.13375)), Evo-0 ([2507.00416](https://arxiv.org/abs/2507.00416)). Severity: high. Active: very (crowding).

**5. Cross-embodiment transfer / negative transfer.** No zero-shot transfer across robots with different action spaces/DoF; mixing heterogeneous embodiments causes *negative transfer* via policy-gradient conflict. Cites: [2511.05936](https://arxiv.org/abs/2511.05936) (#5), cross-embodiment offline RL ([2602.18025](https://arxiv.org/abs/2602.18025)), Align-Then-Steer. Severity: high. Active: moderate–high.

**6. Inference latency / on-device real-time.** 7B VLAs run ~3–5 Hz on edge vs 20–30 Hz needed; AR decode is memory-bound. Cites: [2511.05936](https://arxiv.org/abs/2511.05936) (#6), VLA-Cache ([2502.02175](https://arxiv.org/abs/2502.02175)), RTC ([2506.07339](https://arxiv.org/abs/2506.07339)). Severity: high (deployment-blocking). Active: very — crowded.

**7. Long-horizon execution & memory.** Good "intent," poor multi-step chaining / memory-dependent tasks. The "good intentions, poor execution" gap is crisp. Cites: INT-ACT ([2506.09930](https://arxiv.org/abs/2506.09930)), MemoryVLA ([2508.19236](https://arxiv.org/abs/2508.19236)), Long-VLA, LoHoVLA. Severity: high. Active: very.

**8. Catastrophic / "spurious" forgetting of VLM priors during action fine-tuning.** Fine-tuning on robot data overwrites vision-text alignment and erodes reasoning/instruction-following — directly feeding problem #2. Cites: VLM2VLA ([2509.22195](https://arxiv.org/abs/2509.22195)), ChatVLA (EMNLP'25). Severity: high. Active: growing.

**9. Failure detection & recovery (just starting).** VLAs map inputs→actions open-loop with no introspection; can't reliably detect own failures. Formalized only in 2025. Cites: SAFE ([2506.09937](https://arxiv.org/abs/2506.09937)), FailSafe ([2510.01642](https://arxiv.org/abs/2510.01642)), Sentinel ([2410.04640](https://arxiv.org/abs/2410.04640)). Severity: high (safety). Active: early — good entry point.

**10. Uncertainty quantification & calibration (very open, safety-critical).** Calibrated task-level uncertainty is an open, safety-critical problem; failure rollouts are low-entropy except brief spikes near onset, so naive mean-aggregation misses them. Cites: Confidence Calibration in VLAs ([2507.17383](https://arxiv.org/abs/2507.17383)), Shifting Uncertainty to Critical Moments ([2603.18342](https://arxiv.org/abs/2603.18342)). Severity: high. Active: nascent — **wide open**.

**11. Safety & adversarial/backdoor vulnerability.** VLAs can be driven to attacker-specified action sequences (targeted backdoor up to 100% on some tasks); no standard guardrails. Cites: AttackVLA, FreezeVLA ([2509.19870](https://arxiv.org/abs/2509.19870)), SafeVLA ([2503.03480](https://arxiv.org/abs/2503.03480)). Severity: high (lower current attention). Active: early.

**12. (Lower-ranked) Whole-body/mobile-manipulation coordination, bidirectional human-robot interaction, multi-agent VLA** — Challenges #7/#10/#9 in [2511.05936](https://arxiv.org/abs/2511.05936); least mature.

## Position papers / surveys (what they name as THE bottleneck)
- *10 Open Challenges Steering the Future of VLA* — [2511.05936](https://arxiv.org/abs/2511.05936): emphasizes **robust reasoning** and **evaluation** as the two most critical.
- *VLA Models: Concepts, Progress, Applications & Challenges* — [2505.04769](https://arxiv.org/abs/2505.04769).
- *What matters in building VLA models (RoboVLMs)* — Nature Machine Intelligence 2024 ([2412.14058](https://arxiv.org/abs/2412.14058)): empirical design study, 600+ experiments.
- *A Survey on Efficient VLA Models* — [2510.24795](https://arxiv.org/abs/2510.24795); *VLA Manipulation systematic review* — [2507.10672](https://arxiv.org/abs/2507.10672).

## Benchmark catalog & known flaws
| Benchmark | Measures | arXiv | Known flaws |
|---|---|---|---|
| **LIBERO** | Lifelong/transfer, 130 tasks, 4 suites | 2306.03310 | **Saturated** (~97–99%); vision-only ≈ language-conditioned (shortcut); doesn't isolate VLM-pretraining value |
| **LIBERO-Plus** | Robustness over 7 perturbation axes | 2510.13626 | It *is* the critique — exposes 95%→<30% drops; a perturbation suite, not a full task set |
| **CALVIN** | Long-horizon language chaining (ABCD) | 2112.03227 | Single embodiment (Franka), sim-only, aging, ABC→D near-saturated |
| **SimplerEnv** | Sim proxy for real (Google-Robot/Fractal, WidowX/Bridge) | 2405.05941 | Only two setups; narrow real task set; per-setup correlation can break; weak contact dynamics |
| **RoboCasa / 365** | Large kitchen sim, 365 tasks incl. memory | 2406.02523 | Sim→real gap; success driven by visual priors (shortcut risk); heavy synthetic-demo reliance |
| **VLABench** | World-knowledge / common-sense / long-horizon, intent-laden language | 2412.18194 | Sim-only; very hard → low absolute scores limit discrimination |
| **RoboArena** | Distributed double-blind *pairwise* real eval on DROID | 2506.18123 | Relative ranking only; needs shared hardware; evaluator-pool dependence |
| **AutoEval** | Autonomous 24/7 real eval w/ learned success classifiers + auto reset | 2503.24278 | Scene/hardware-limited; classifier/reset errors bound fidelity |
| **RobotArena∞** | Real→sim translation → scalable sim eval + preference | 2510.23571 | Depends on real-to-sim fidelity; new/unvalidated |

**Cross-cutting flaws:** (a) sim→real gap (contact/lighting); (b) **saturation** (LIBERO/CALVIN); (c) **validity failure** (high scores ≠ competence; language ignored); (d) **unfair comparison** (non-standard per-lab real setups, tokenizer diffs); (e) **reproducibility & cost** of real-robot eval; (f) **metric weakness** (binary/2D success misses 3D structure).

## Is there a recognized reproducibility problem? — Yes, unambiguously.
It is (1) named the top-2 bottleneck in the leading position paper, (2) the explicit motivation for an entire 2025–26 benchmark wave (RoboArena, AutoEval, RobotArena∞, RADAR, VLA-REPLICA), and (3) empirically validated by robustness audits (LIBERO-Plus, LangGap). **No standard has won** — the door is open, but the door is *crowded* (see [research log](../research-log.md): the eval-benchmark idea was killed precisely because ≥6 groups arrived in ~8 weeks).
