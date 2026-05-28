# VLA Landscape (2023–2026)

[← Knowledge Base](README.md) · [← Repo README](../../README.md)

A precise map of the major Vision-Language-Action models, where the field is converging, what is already saturated, and where genuine gaps remain. Compiled from primary arXiv sources + the two key 2025 surveys (*A Survey on Efficient VLA Models*, [2510.24795](https://arxiv.org/abs/2510.24795); *10 Open Challenges Steering the Future of VLA*, [2511.05936](https://arxiv.org/abs/2511.05936)).

## 1. Model catalog

**Action representation legend:** TOK = discretized action tokens · FM = flow-matching action expert · DIFF = diffusion head · DCT = FAST frequency tokenization · L1 = continuous regression.

### Foundational / discrete-token era
| Model | Year · Group | Base / action | Data · Open? | Headline → stated limitation | arXiv |
|---|---|---|---|---|---|
| **RT-1** | 2022 · Google | EfficientNet+Tx · TOK(256) | 130k proprietary · open wts | Scalable real multitask → no web semantics, single-embodiment | 2212.06817 |
| **RT-2** | 2023 · Google DM | PaLI-X/PaLM-E 5–55B · text-TOK | proprietary · **closed** | Emergent semantic generalization → 1–5 Hz latency, no new physical skills | 2307.15818 |
| **Open X-Embodiment / RT-X** | 2023 (CoRL) · 60+ insts | *dataset* 1M+ traj, 22 embodiments | open | Positive cross-embodiment transfer → heterogeneous action spaces unmerged | 2310.08864 |

### Flow-matching / diffusion action-expert era (current mainstream)
| Model | Year · Group | Base / action | Open? | Headline → limitation | arXiv |
|---|---|---|---|---|---|
| **Octo** | 2024 · Berkeley | Tx + DIFF head (27/93M) | full open | Open generalist → small capacity, weak language | 2405.12213 |
| **OpenVLA** | 2024 · Stanford+ | Prismatic(Llama2+DINO/SigLIP) 7B · TOK | full open | Beats RT-2-X 55B w/ 7× fewer params → slow AR decode (~6 Hz), single-image | 2406.09246 |
| **π₀** | 2024 · Physical Intelligence | PaliGemma + FM expert (~3.3B) | wts open (openpi) | Dexterous long-horizon (laundry) → costly expert, big proprietary data | 2410.24164 |
| **π₀-FAST / FAST** | 2025 · PI | FAST = DCT+BPE action tokens | tokenizer open | AR VLA at high freq, 5× faster train than DIFF → AR slower at inference | 2501.09747 |
| **π₀.5** | 2025 · PI | π₀ + heterogeneous co-train | — | First to clean novel kitchens → in-the-wild robustness still open | 2504.16054 |
| **π\*0.6 / RECAP** | Nov 2025 · PI | + RL from deployment + Knowledge Insulation | — | 13 h autonomous espresso; "RL is back" → very new, data/compute heavy | 2511.14759 |
| **RDT-1B** | 2024 (ICLR'25) · Tsinghua | DIFF-Tx 1.2B, unified bimanual action space | open | Largest DIFF VLA at release → bimanual-focused, heavy data | 2410.07864 |
| **CogACT** | 2024 · Microsoft | VLM + dedicated DIFF action Tx | open | +35% sim / +55% real over OpenVLA → added module complexity | 2411.19650 |
| **OpenVLA-OFT** | 2025 · Stanford | parallel decode + chunking + **L1** + bidir attn | open | LIBERO 76.5→**97.1%**, **26× throughput**; beats π₀/RDT on ALOHA → a *recipe*, gains mostly in-domain | 2502.19645 |
| **GR00T N1** | 2025 · NVIDIA | dual-system: VLM S2 + **FM** S1 (humanoid) | open | Humanoid foundation → needs embodiment-specific heads, sim-to-real gated | 2503.14734 |
| **Gemini Robotics / -ER** | 2025 · Google DM | Gemini 2.0 + 50 Hz local decoder | **closed** | <160 ms cloud backbone + on-robot action → closed, cloud dependency | 2503.20020 |
| **Gemini Robotics 1.5 + ER 1.5** | Sep 2025 · DM | agentic dual-stack, reasoning traces during exec | VLA closed | Cross-embodiment skill transfer → VLA access restricted, safety open | 2510.03342 |
| **Helix** | Feb 2025 · Figure | dual-system S2 7–9 Hz / S1 200 Hz 35-DoF | **closed, blog only** | First VLA on two cooperating robots → **no peer-reviewed/arXiv paper; claims unverified** | (blog) |

### Spatial / efficient / data-efficient cluster
| Model | Year · Group | Idea | Open? | Note | arXiv |
|---|---|---|---|---|---|
| **Magma** | 2025 · Microsoft | Set-of-Mark + Trace-of-Mark from video | open | Digital+physical agent → low-freq control | 2502.13130 |
| **LAPA** | 2024 (ICLR'25 oral) | Latent action pretraining from *actionless* video (VQ-VAE) | open | Beats OpenVLA on some real tasks w/o action labels → lossy latents | 2410.11758 |
| **SpatialVLA** | 2025 · Shanghai AI Lab | Ego3D position encoding + adaptive action grids | open | re-discretizable per robot → depth dependence | 2501.15830 |
| **TinyVLA** | 2024 | 70M–1.4B + DIFF, no pretraining | open | +25.7% over OpenVLA, 5.5× fewer params, ~20× lower latency | 2409.12514 |
| **SmolVLA** | 2025 · HuggingFace | 450M, layer-skip + FM expert, async inference | open | Trained on 487 community datasets → **0% on adversarial zero-shot cross-embodiment (RoboGate)** | 2506.01844 |

### Reasoning-augmented cluster (filling fast)
| Model | Year | Idea | Note | arXiv |
|---|---|---|---|---|
| **ECoT** | 2024 · Berkeley | Embodied chain-of-thought before action | +28% over OpenVLA, no extra robot data → reasoning slows inference | 2407.08693 |
| **CoT-VLA** | 2025 · NVIDIA/Stanford | Visual CoT: predict future-frame subgoal then act | +17% real / +6% sim → image subgoal expensive | 2503.22020 |
| **MemoryVLA** | 2025 | Cognition-Memory-Action bank for long-horizon | beats CogACT & π₀ (+26 long-horizon) | 2508.19236 |

### Force/tactile & world-model-RL clusters (emerging, hot)
- **Force/tactile:** ForceVLA (force-aware MoE, [2505.22159](https://arxiv.org/abs/2505.22159)), Tactile-VLA ([2507.09160](https://arxiv.org/abs/2507.09160)), FAVLA fast-slow force ([2602.23648](https://arxiv.org/abs/2602.23648)). Open issue: vision tokens swamp force tokens; sensor-rate mismatch (15 Hz vision vs 100–1000 Hz F/T) unsolved.
- **World-model RL (2025–26):** WMPO ([2511.09515](https://arxiv.org/abs/2511.09515)), VLA-RFT ([2510.00406](https://arxiv.org/abs/2510.00406)), RLinf-VLA ([2510.06710](https://arxiv.org/abs/2510.06710)). Theme: post-train VLAs with RL inside learned/video world models. Open: pixel-fidelity, compounding error under sparse reward.

## 2. Convergence map (where the field is heading)
1. **Flow-matching / diffusion action experts on a (frozen-ish) VLM are the default** for high-freq continuous control (π₀, π₀.5, GR00T, RDT, CogACT, SmolVLA). Naive 256-bin token discretization is being abandoned except where FAST/DCT revives AR.
2. **Dual-system "System-2 reasoning + System-1 control"** is the convergent macro-architecture (GR00T, Helix, Gemini 1.5, Hi Robot). Decoupled frequencies (~7–10 Hz semantic, 50–200 Hz motor).
3. **Parallel / non-autoregressive decoding + action chunking** for throughput (OpenVLA-OFT 26×, PD-VLA, async inference).
4. **Hierarchical planner + policy with explicit reasoning traces** (ECoT, CoT-VLA, Hi Robot, Gemini-ER).
5. **Data convergence on OXE + human video + synthetic + web VQA**; latent-action pretraining from actionless video (LAPA, GR00T, EgoVLA).
6. **Efficiency-by-construction**: sub-1B backbones, layer-skip/pruning, quantization (down to 1-bit BitVLA), KV-cache/token pruning — a whole sub-field.

## 3. Crowded / saturated (avoid claiming as novel)
- "Add a diffusion/flow-matching action expert to a VLM" — commoditized since π₀.
- Generic dual-system fast/slow architecture — every major lab.
- Parallel decoding / action chunking for speed — OpenVLA-OFT + a dozen 2025 papers.
- Naive discrete action tokenization + incremental fixes — superseded.
- Yet-another smaller/faster VLA (TinyVLA, SmolVLA, MiniVLA, EdgeVLA, NORA, BitVLA…) — extremely crowded.
- LIBERO / SimplerEnv leaderboard-chasing — near-saturated (OFT 97.1%).
- Single-image single-arm tabletop manipulation as the headline setting.
- Embodied chain-of-thought as a generic add-on — multiple variants exist.

## 4. The open gaps (paper-worthy as of mid-2026)
1. **Zero-shot cross-embodiment action transfer** — still the hardest unsolved bottleneck (SmolVLA reports 0% on adversarial zero-shot; action heterogeneity flagged in [2511.05936](https://arxiv.org/abs/2511.05936)).
2. **Tightly-coupled high-rate force/tactile fusion inside the action expert** — vision dominates attention; sensor-rate mismatch unsolved (→ Proposal #1 Bandwidth-Frontier).
3. **On-robot model-free-ish RL that self-improves without a fragile world model** — π\*0.6/RECAP showed RL "works," but sample-efficiency/safety open.
4. **Calibrated uncertainty + failure detection + asking for help** — safety/HRI underdeveloped.
5. **Whole-body loco-manipulation as a unified VLA with long-horizon memory** — most flagships are arm-only/tabletop.

## Caveats
- **Helix has no peer-reviewed/arXiv paper** (company blog only) — treat performance claims as unverified.
- The efficiency survey [2510.24795](https://arxiv.org/abs/2510.24795) contains metadata errors (e.g., GR00T mis-attributed) — corrected above from primary sources; don't cite its summary table uncritically.
- Worth citing directly in related work: efficient-VLA survey [2510.24795](https://arxiv.org/abs/2510.24795), 10-challenges [2511.05936](https://arxiv.org/abs/2511.05936), VLA manipulation systematic review [2507.10672](https://arxiv.org/abs/2507.10672), and the design study *RoboVLMs / "What matters in building VLA models"* (Nature Machine Intelligence 2024, [2412.14058](https://arxiv.org/abs/2412.14058)).
