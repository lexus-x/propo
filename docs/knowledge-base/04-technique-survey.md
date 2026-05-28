# Technique Survey — Cross-Domain Win-Win Scorecards

[← Knowledge Base](README.md) · [← Repo README](../../README.md)

The full cross-domain sweep: for each candidate technique, the closest existing VLA work, how saturated it is, and an honest **win-win scorecard** — **N**ovel · **B**etter performance · **C**heaper (less compute/data) · **F**aster (`✓`/`~`/`✗`/`?`). This is the raw intelligence behind which ideas became proposals and which were killed.

> The recurring lesson: nearly every *hyped* technique was already ported to VLA within ~2 months of appearing. Genuine white space concentrated in (a) **cross-disciplinary imports** from less-hyped fields and (b) **characterization studies**.

---

## A. Inference-time compute & reasoning
| Technique | Closest VLA work | Sat. | N·B·C·F | Verdict / remaining angle |
|---|---|---|---|---|
| Test-time scaling (best-of-N + verifier) | RoboMonkey 2506.17811 · RoVer 2510.10975 · MG-Select 2510.05681 · V-VLAPS 2601.00969 | HIGH | ✓·✓·✗·✗ | Fails win-win (more compute, slower). Uncertainty-*gated* allocation (LATTS analogue 2509.20368) was open but later closed by SCALE/VLA-ATTC |
| World-model as online verifier ("imagine-and-rank") | WorldVLA 2506.21539 · DreamVLA 2507.04447 · Self-Correcting VLA 2602.21633 | HIGH | ?·✓·✗·✗ | World models used as co-training/RL-in-imagination, not test-time MPC ranking — but compute-heavy |
| Latent / continuous-thought reasoning (Coconut→VLA) | Latent-Reasoning-VLA 2602.01166 · RD-VLA 2602.07845 · ThinkAct 2507.16815 | HIGH | ✗ | Taken |

## B. RL post-training
| Technique | Closest VLA work | Sat. | N·B·C·F | Verdict |
|---|---|---|---|---|
| GRPO family / R1 recipes | SimpleVLA-RL 2509.09674 · RIPT-VLA 2505.17016 · VLA-RL 2505.18719 · TGRPO 2506.08440 | HIGH | ✗ | Done; and **PPO>GRPO** for VLAs ([2505.19789](https://arxiv.org/abs/2505.19789)) |
| **Verifier-free intrinsic-reward RL** (Intuitor 2505.19590, TTRL 2504.16084 → VLA) | none on *VLA training* (closest TOPReward 2602.19313) | LOW | ✓·?·~·~ | Best RL survivor: attacks the reward bottleneck, no reward-model cost — but RL ⇒ more train compute; confidence≠correctness risk |
| GSPO sequence/chunk-level optimization (2507.18071) | action-chunk PPO 2509.25718 | LOW | ✓·?·✗·✗ | Architecturally apt for action chunks, but only 2/4 (training-only); must beat PPO |
| ProRL prolonged RL (2505.24864) | SimpleVLA-RL pushcut emergent skill | MED | ?·✓·✗✗·✗ | Boundary-expansion shown; much more compute |
| Process reward models / step rewards | Robo-Dopamine 2512.23703 · VLA-RL 2505.18719 · RoVer 2510.10975 | HIGH | ✗ | Saturated; PRM training expensive |

## C. Generative action heads
| Technique | Closest VLA/policy work | Sat. | N·B·C·F | Verdict |
|---|---|---|---|---|
| Flow-map / Align-Your-Flow (2506.14603, Decoupled-MeanFlow 2510.24474) | **SnapFlow 2604.05656** · MeanFlow-VLA 2603.01469 | HIGH | ✗ | **Killed** — method pre-published; 1-NFE already 98.75% LIBERO above teacher |
| MeanFlow / shortcut one-step | MP1 2507.10543 · SnapFlow 2604.05656 | HIGH | ✗ | Scooped |
| Consistency / CTM distillation | Consistency Policy 2405.07503 · CEED-VLA 2506.13725 | HIGH/MED | ✗ | CEED-VLA covers AR-VLA case |
| **Energy-Based Transformer head** (2507.02092) | **EBT-Policy 2510.27545** | LOW(fresh) | ✓·✓·✓·✓ | **Killed by red-team** — all claims pre-claimed by EBT-Policy; energy≠calibrated (2107.08785). Salvage: multimodality study only |
| Bayesian Flow Networks (2308.07037) | only Guided-BFNs (planning) — none in VLA | very LOW | ✓·?·?·? | Cleanest *unclaimed* head, but zero robotics priors → highest research risk |
| Discrete / masked diffusion actions | Discrete Diffusion VLA 2508.20072 · LLaDA-VLA 2509.06932 | HIGH | ✗ | Saturated |

## D. Self-supervised representations & world models
| Technique | Closest VLA work | Sat. | N·B·C·F | Verdict |
|---|---|---|---|---|
| **V-JEPA 2** as VLA *encoder* (2506.09985) | JEPA-VLA 2602.11832 · VLA-JEPA 2602.10098 | HIGH(Feb'26) | ✗ | Encoder injection scooped within weeks |
| V-JEPA **2-AC** as language-conditioned *planner* | original defers language to future work | LOW | ✓·✓·✓·✗ | Planner route still open; MPC = slow |
| DINOv3 → OpenVLA fused encoder swap | DINOv3-Diffusion-Policy 2509.17684 | LOW | ✓·~·?·= | Clean but incremental; DINOv3 non-Apache license |
| DINO-WM world model (2411.04983) | none as language VLA | LOW | ✓·✓·✓·✗ | Open but planning loop slow |
| NVIDIA Cosmos Policy | itself (Jan'26) | MED | ?·✓·✓✓·✗ | RoboCasa 67% w/ 50 demos vs π₀ 62.5% w/ 300 — strong data-efficiency, vendor-driven |
| VGGT geometry backbone | GeoAware-VLA 2509.14117 · VGGT-DP 2509.18778 · VGA 2604.12908 | HIGH | ✗·✓·?·✗ | Saturated; strongest *counter-argument* to JEPA (geometry wins OOD viewpoints) |

## E. Systems efficiency
| Technique | Closest VLA work | Sat. | N·B·C·F | Verdict / headline |
|---|---|---|---|---|
| **NVFP4 / MXFP4** hardware-native FP4 | none (all VLA quant is integer: QVLA 2602.03782, BitVLA 2506.07530) | ~0 | ✓·?·✓·✓ | Clean gap; "~2–4× throughput, 4× memory, <1% drop" — but **Blackwell-gated** |
| Mixture-of-Depths token routing (2404.02258) | DeeR-VLA 2411.02359 · MoLe-VLA 2503.20384 (network-level) | LOW(token) | ✓·~·✓·✓ | **Killed by red-team** — VLA-Pruner 2511.16449 / LightVLA 2509.12594 own the numbers; pruning hurts VLA |
| KV-cache *quantization* (KIVI 2402.02750, KVQuant 2401.18079) | KV-Efficient-VLA 2509.21354 (eviction) | LOW | ✓·✓·✓·? | Orthogonal to eviction, but VLA contexts short → small payoff |
| Speculative decoding (EAGLE-3 / Medusa) | Spec-VLA 2507.22424 (EAGLE-1) · HeiSD | MED | ?·✓·✗·✓ | Lossless latency; no compute win; weak vs parallel decoders |
| 2:4 semi-structured sparsity | structured pruning only | LOW | ?·?·✓·✓ | Best as a *multiplier* on NVFP4, accuracy-fragile |

## F. Backbones / sequence architectures
| Technique | Closest VLA work | Sat. | N·B·C·F | Verdict |
|---|---|---|---|---|
| Gated DeltaNet (2412.06464) | none (RoboMamba=Mamba-1 2406.04339) | ~0 | ✓·?·✓·✓ | Novel backbone; **but no linear backbone has beaten a strong Tx-VLA at matched scale** |
| Test-Time-Training layers (2407.04620) | none as VLA backbone | ~0 | ✓·?·?·? | Exciting (in-episode learning) but inner-loop may erase speed win |
| Mamba-3 / RWKV-7 / Titans | RoboMamba/MambaVLA (Mamba-1/2) | LOW–MED | varies | Family already trodden; perf risk is the bar |
| Hybrid Mamba-2/Tx (Jamba/Hymba) | MaTVLM 2503.13440 (VLM only) | LOW | ?·✓·✓·✓ | Safest "match Tx at lower cost"; application-novel only |

## G. Cross-disciplinary imports (where the survivors came from)
| Technique | Closest VLA work | Sat. | N·B·C·F | Outcome |
|---|---|---|---|---|
| **Iterative Learning Control** (training-free residual corrector) | none (Task-Level ILC 2602.21302 is non-VLA) | LOW | ✓·✓·✓·✓ | **→ Proposal #7** |
| **Central Pattern Generator** action head | none (LMP 2602.02839, FODMP 2603.24806 are DMP) | LOW | ✓·✓·✓·✓ | **→ Proposal #5** |
| Koopman linearization of VLA latents | Koopman on *plant* only (2511.06515) | LOW–MED | ✓·✓·✓·✓ | Honorable mention (latent non-stationarity risk) |
| MPC over action chunks | adaptive chunking 2604.04161 (not MPC) | LOW | ✓·✓·~·~ | Alive, harder to execute |
| CBF safety filter | VLSA/AEGIS 2512.11891 · SafeVLA 2503.03480 | HIGH | ✗ | Killed (scooped) |
| Variable-rate / RD action VQ | ActionCodec 2602.15397 · Compression-Gap 2604.03191 · OAT 2602.04215 | HIGH | ~ | Scoop-adjacent |
| Channel-capacity asymmetric token allocation | VLA-Pruner 2511.16449 · OneWM-VLA 2605.07931 | MED | ✓·✓·✓·✓ | Risky (pruning crowd) |
| **Self-paced curriculum** (SFT demo ordering) | none (STARE-VLA 2512.05107 is RL stage-curriculum) | LOW | ✓·?·✓·✓ | **→ Proposal #6** |
| **Depth distillation** (RGB-only deploy) | QDepth-VLA 2510.14836 (training-depth) · DepthVLA 2510.13375 | HIGH | ~·✓·~·= | **→ Proposal #8** (scoop-adjacent, flagged) |
| Cerebellar / forward-model anticipatory corrector | FAVLA 2602.23648 (reactive) | LOW | ✓·✓·✓·✓ | Honorable mention (variant of #1) |
| DMP / ProDMP action parameterization | LMP 2602.02839 · FODMP 2603.24806 | HIGH | ✗ | Killed |
| Spiking-NN action loop | Spiking Diffusion Policy 2409.11195 | HIGH | hw-only | Killed (neuromorphic-HW dependent) |
| Mixture-of-LoRA-experts | AdaMoE · ChatVLA · DriveMoE · FedVLA · MoE-ACT | HIGH | ✗ | Killed |
| Self-distillation / Born-Again | ActDistill · MoLe-VLA 2503.20384 · AC²-VLA 2601.19634 | HIGH | ✗ | Killed |
| Federated / continual VLA | FedVLA | HIGH | ✗ | Killed |
| MAML meta-learning | superseded by MoS-VLA (gradient-free 1-shot) | MED | ✗ | Killed |
| Augmentation theory (SE(3) sample-complexity bounds) | RoCoDA (empirical only) | LOW | ✓·✓·✓·? | Honorable mention (theory-heavy) |
| Off-policy critic for diffusion VLA | ALOE 2602.12691 (AR-VLA) | MED | ✓·✓·✗·✗ | Author-flagged (π\*0.6) but RL ⇒ more compute |
| Counterfactual physics-consistent synthetic data | InternData-A1 2511.16651 (scale, not counterfactual) | MED | ✓·✓·✓·= | Author-flagged (GR00T); data-genre crowded |
| Persistent instruction-KV cache (cross-rollout) | OxyGen (within-episode) · VLA-Cache 2502.02175 | LOW | ✓·=·✓·✓ | Honorable mention ("just prompt caching"; short fuse) |
| Model merging / mech-interp steering | MergeVLA 2511.18810 · Mech-Interp-Steering 2509.00328 | MED | ✗ | Killed |

## The connective insight
**Calibrated uncertainty** links the least-saturated, most-feasible angles (test-time compute allocation, ask-for-help, replanning triggers, verified flywheel labeling, certified safety) — but every *standalone* use was being filled in 2025–26. The durable opportunities turned out to be the **cross-disciplinary fast-inner-loop / structured-action ideas** (Proposals #1, #5, #7) and **characterization studies** (#2, #3), not another bolt-on to the saturated method stack.
