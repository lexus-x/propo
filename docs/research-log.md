# Research Log — The Killed Ideas

[← Back to README](../README.md)

~40 candidate ideas were generated across multiple brainstorming and cross-disciplinary search rounds; ~16 were killed by adversarial red-teams and live scoop-checks against the 2025–2026 arXiv literature. This is the full record — transparency about what did NOT survive, and why.

---

## Section 1 — Killed by the 4 adversarial red-teams (leading method ideas)

| Killed idea | Why it died / closest scooping work | arXiv IDs |
|---|---|---|
| **EBT-VLA** — energy-based action head replacing the diffusion/flow head | Pre-empted by EBT-Policy; energy-as-uncertainty refuted by EBM-OOD literature; test-time-verification niche already owned by SAFE and the composable-constraints cluster; OpenVLA-OFT's L1 head already ~97% on LIBERO, leaving no headroom | [2510.27545](https://arxiv.org/abs/2510.27545), [2107.08785](https://arxiv.org/abs/2107.08785), [2602.12281](https://arxiv.org/abs/2602.12281) |
| **MoD-VLA** — token-level Mixture-of-Depths for VLA inference efficiency | VLA-Pruner matches the exact efficiency numbers; LightVLA owns the vision-token-sparsity analysis; naive token pruning demonstrably hurts VLAs (FastV/ToMe show −14 to −20% task success) | [2511.16449](https://arxiv.org/abs/2511.16449), [2509.12594](https://arxiv.org/abs/2509.12594) |
| **Calibration-free Equivariant VLA** — SE(3)-equivariant policy head with no per-scene calibration | Functionally equal to Eq.Bot ⊕ E2VLA; the calibration gap is being closed equivariance-free by AnyCamVLA and the VGGT cluster; canonicalization actively fights pretrained VLM priors | [2511.15194](https://arxiv.org/abs/2511.15194), [2509.14630](https://arxiv.org/abs/2509.14630), [2603.05868](https://arxiv.org/abs/2603.05868), [2310.01647](https://arxiv.org/abs/2310.01647) |
| **Flow-Map / Align-Your-Flow action head** — rectified-flow with learned coupling map for 1–2 step inference | Method pre-published in Decoupled MeanFlow and Align-Your-Flow; the remaining differentiator pre-empted by SnapFlow; LIBERO saturated as a benchmark | [2510.24474](https://arxiv.org/abs/2510.24474), [2506.14603](https://arxiv.org/abs/2506.14603), [2604.05656](https://arxiv.org/abs/2604.05656) |

---

## Section 2 — Killed in the first cut (failed the win-win test or already crowded)

| Killed idea | Why it died / closest scooping work | arXiv IDs |
|---|---|---|
| **Test-time scaling / introspective controller** | Adds compute, not speed; niche saturated by RoboMonkey, MG-Select, AC²-VLA | [2506.17811](https://arxiv.org/abs/2506.17811), [2510.05681](https://arxiv.org/abs/2510.05681), [2601.19634](https://arxiv.org/abs/2601.19634) |
| **RL post-training** (intrinsic-reward, GSPO, ProRL variants) | Training-time cost unacceptable for the speed axis; PPO > GRPO finding undermines the recipe; SimpleVLA-RL already open-sourced the full pipeline | [2505.19789](https://arxiv.org/abs/2505.19789), [2509.09674](https://arxiv.org/abs/2509.09674) |
| **One-step flow distillation** | Directly scooped by SnapFlow and MeanFlow-VLA | [2604.05656](https://arxiv.org/abs/2604.05656), [2603.01469](https://arxiv.org/abs/2603.01469) |
| **SSM / linear backbones** (Gated DeltaNet, TTT, Mamba-3) | Performance risk vs. transformer; RoboMamba precedent shows diminishing returns on manipulation benchmarks | [2406.04339](https://arxiv.org/abs/2406.04339) |
| **V-JEPA-2 encoder injection** | Directly scooped by JEPA-VLA and VLA-JEPA | [2602.11832](https://arxiv.org/abs/2602.11832), [2602.10098](https://arxiv.org/abs/2602.10098) |
| **NVFP4 quantization for VLAs** | Blackwell-hardware-gated; application-only novelty; integer-only VLA quantization already published | [2602.03782](https://arxiv.org/abs/2602.03782) |
| **DINOv3 encoder swap** | Incremental over existing encoder-swap work; no benchmark differentiation |  |
| **Neuro-symbolic / active-inference / Hopfield-memory** | Fails the "faster" axis; no manipulation-benchmark wins demonstrated in any comparable work |  |
| **Model-merging / mech-interp steering** | Scooped by MergeVLA and Mechanistic-Interpretability-Steering | [2511.18810](https://arxiv.org/abs/2511.18810), [2509.00328](https://arxiv.org/abs/2509.00328) |

---

## Section 3 — Killed in the validation round

| Killed idea | Why it died / closest scooping work | arXiv IDs |
|---|---|---|
| **Eval / robustness benchmark** | Most-scooped corner of the space; at least six concurrent benchmarks published | [2602.17659](https://arxiv.org/abs/2602.17659), [2603.00592](https://arxiv.org/abs/2603.00592), [2510.13626](https://arxiv.org/abs/2510.13626), [2506.09930](https://arxiv.org/abs/2506.09930), [2603.13966](https://arxiv.org/abs/2603.13966), [2602.10980](https://arxiv.org/abs/2602.10980) |
| **Multi-rate force/tactile fast-slow control** (as standalone headline) | TIDAL, FAVLA, DuoCore-FS, AT-VLA collectively occupy this space; survived only as the reframed "control-bandwidth frontier" in Proposal #1 | [2601.14945](https://arxiv.org/abs/2601.14945), [2602.23648](https://arxiv.org/abs/2602.23648), [2512.20188](https://arxiv.org/abs/2512.20188), [2605.07308](https://arxiv.org/abs/2605.07308) |
| **Data-centric coreset as a headline** | CUPID, FT-NCFM, QoQ saturate the short-list framing; survived only as the OXE-scale selection scaling-law in Proposal #4 | [2506.19121](https://arxiv.org/abs/2506.19121), [2511.16233](https://arxiv.org/abs/2511.16233), [2603.09056](https://arxiv.org/abs/2603.09056) |
| **Cross-embodiment gradient-surgery** | Scooped by the exact paper flagged during the scoop-check; VLA-Forget provides a complementary angle already | [2602.18025](https://arxiv.org/abs/2602.18025), [2604.03956](https://arxiv.org/abs/2604.03956) |
| **KV-reuse / "beneficial sparsity"** | VLA-Cache, NoTVLA, and AC²-VLA collectively own the token-reuse / KV-cache niche | [2502.02175](https://arxiv.org/abs/2502.02175), [2510.03895](https://arxiv.org/abs/2510.03895), [2601.19634](https://arxiv.org/abs/2601.19634) |

---

## Section 4 — Killed in the cross-disciplinary hunt

| Killed idea | Why it died / closest scooping work | arXiv IDs |
|---|---|---|
| **CBF safety filter on VLA outputs** | VLSA/AEGIS and SafeVLA already implement this pattern | [2512.11891](https://arxiv.org/abs/2512.11891), [2503.03480](https://arxiv.org/abs/2503.03480) |
| **DMP action parameterization** | Language Movement Primitives and FODMP cover the parameterization angle | [2602.02839](https://arxiv.org/abs/2602.02839), [2603.24806](https://arxiv.org/abs/2603.24806) |
| **Efference-copy fast/slow reflex** | FAVLA and DuoCore-FS implement this pattern; overlaps with Proposal #1 origin | [2602.23648](https://arxiv.org/abs/2602.23648), [2512.20188](https://arxiv.org/abs/2512.20188) |
| **Spiking-neural-net action loop** | Spiking Diffusion Policy establishes precedent; hardware-dependent deployment makes benchmark comparison infeasible | [2409.11195](https://arxiv.org/abs/2409.11195) |
| **Rate-distortion / IB action tokenization** | ActionCodec, Compression Gap, OAT, and BC-IB collectively saturate the information-bottleneck framing for action representations | [2602.15397](https://arxiv.org/abs/2602.15397), [2604.03191](https://arxiv.org/abs/2604.03191), [2602.04215](https://arxiv.org/abs/2602.04215), [2502.02853](https://arxiv.org/abs/2502.02853) |
| **Uncertainty-gated adaptive compute** | SCALE, VLA-ATTC, and A3 occupy this niche |  |
| **Speculative decoding of action chunks** | HeiSD, A3, and Spec-VLA already publish this | [2507.22424](https://arxiv.org/abs/2507.22424) |
| **Training-free smoothing / jerk reduction** | ACG and TACO cover post-hoc smoothing; no differentiated benchmark claim available |  |
| **Mixture-of-LoRA-experts** | AdaMoE, ChatVLA, DriveMoE, FedVLA, MoE-ACT collectively saturate this design space |  |
| **Self-distillation / Born-Again VLA** | ActDistill, MoLe-VLA, and AC²-VLA implement this pattern | [2503.20384](https://arxiv.org/abs/2503.20384), [2601.19634](https://arxiv.org/abs/2601.19634) |
| **Federated / continual VLA** | FedVLA occupies federated; MAML meta-learning superseded by MoS-VLA |  |

---

## Section 5 — Honorable mentions (survived a hunt but not promoted to a full proposal)

These ideas were not killed outright but failed to reach the bar for a standalone proposal — either too narrow, too dependent on a surviving proposal, or flagged as likely scooped within 6 months.

| Idea | Note |
|---|---|
| **Koopman-operator linearization of VLA latents** | Theoretically clean; no clear manipulation benchmark win |
| **Cerebellar / forward-model anticipatory corrector** | A variant of Proposal #1's fast-slow control; merged into that framing rather than promoted separately |
| **Persistent instruction-KV cache (PIKB)** — cross-rollout prompt caching | Partially addressed by KV-reuse cluster above; the cross-rollout angle is narrow |
| **Off-policy critic for diffusion VLAs** | Flagged by π*0.6 / RECAP; collapses to RL post-training → more compute, fails the speed axis | [2511.14759](https://arxiv.org/abs/2511.14759) |
| **Counterfactual physics-consistent synthetic data** | Flagged by GR00T N1; data-centric framing merges into Proposal #4 scope | [2503.14734](https://arxiv.org/abs/2503.14734) |

---

> **Note:** arXiv IDs sourced from automated search agents — verify all links before citing; snapshot May 2026.
