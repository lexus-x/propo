# Project 8 — DepthFree-VLA: Train-Time Depth Distillation for RGB-Only Deployment

---

## ⚠️ Scoop Status

**Verdict: LIKELY SCOOPED (partially) by QDepth-VLA — with a defensible, narrow delta remaining.**

**Exact overlap with QDepth-VLA (arXiv 2510.14836):**
QDepth-VLA explicitly uses depth supervision at training only and deploys with RGB only. The paper states: *"avoiding the need for additional sensory inputs during inference."* It uses Video-Depth-Anything (ViDA) — a monocular video depth estimator — to generate pseudo-depth maps from RGB frames in OXE/LIBERO, trains a dedicated depth expert to predict VQ-VAE quantized depth tokens as an auxiliary objective, and drops that expert entirely at deployment. The VLA backbone (π0 + PaliGemma-3B / SigLIP) absorbs 3D geometric priors through the shared vision encoder. This is the core claim of the proposed DepthFree-VLA.

**What QDepth-VLA does NOT do (the defensible delta):**
1. QDepth-VLA adds a **separate branch depth expert** that sits beside the vision encoder. It does not inject depth knowledge *into* the vision encoder via a bottleneck adapter; depth tokens never pass through a constrained representation layer within the backbone. The proposed depth-bottleneck adapter architecture is architecturally distinct.
2. QDepth-VLA is **specific to a single VLA family** (π0 / PaliGemma). It does not demonstrate portability to OpenVLA, RoboVLMs, or other diverse backbones. A "plug-in-to-any-VLA" framing remains open.
3. QDepth-VLA uses **Video-Depth-Anything (ViDA)**; Depth Anything V2 (image-level) as teacher and the distillation specifically into the backbone representation is not demonstrated.
4. **MonoLift** (NeurIPS 2025, OpenReview wZzC5rpDY1) also covers depth-to-RGB-student distillation but at the non-VLA policy level (non-language-conditioned), using tri-level distillation (spatial/temporal/action) rather than vision encoder bottleneck adaptation.
5. **UniLACT** (arXiv 2602.20231, Feb 2026) distills depth into latent *action* representations (not the vision encoder backbone), requiring a multi-stage pretraining pipeline rather than a lightweight adapter.

**Recommended pivot if submitting:** Frame the contribution as **plug-in depth-bottleneck adapter for any frozen VLA vision encoder**, with Depth Anything V2 as teacher. The key differentiator is (a) backbone-internal depth bottleneck (not a separate branch), (b) adapter-only parameter overhead (~2–5 M params, no retraining the LLM), and (c) cross-VLA generalization study (OpenVLA, π0, RoboVLMs). This is narrower and harder to claim, but it is genuinely unoccupied as of May 2026.

---

## TL;DR

Depth sensors at deployment are expensive and fragile, but monocular depth models can generate pseudo-depth at training time for free. This proposal distills depth knowledge from Depth Anything V2 into a lightweight bottleneck adapter inside a frozen VLA vision encoder, so the model gains 3D spatial grounding but requires only RGB at deployment. The core scoop risk — RGB-only deployment with training-time depth distillation — is already claimed by QDepth-VLA (Oct 2025); the defensible contribution is the backbone-internal bottleneck adapter design and cross-architecture portability, which QDepth-VLA does not address.

---

## The Gap / Motivation

The VLA survey "10 Open Challenges" (arXiv 2511.05936, Nov 2025) identifies depth sensing as a persistent limitation: models such as MolmoAct and SpatialVLA that impute depth from RGB are *"limited to the training phase where the VLA explicitly learns to impute depth from the RGB frames … This naturally eliminates the need for a specialized depth-gauging camera, but the limitation still remains due to the above reasons."* The survey flags the gap between training-time geometric awareness and deployment-time sensory constraints as an unsolved challenge.

In practice, depth cameras (RealSense, ZED) cost $100–$400, require calibration, fail under direct light, and are often absent on commodity robots. Meanwhile, monocular depth foundation models (Depth Anything V2, ViDA) can generate dense metric-scale pseudo-depth from any RGB frame at no additional hardware cost. The open question is not whether pseudo-depth is useful as a training signal — QDepth-VLA and MonoLift confirm it is — but *how* to inject that signal into a frozen VLA backbone as a portable, lightweight add-on without re-training the full LLM.

---

## Precise Novelty & Positioning

| Paper | arXiv ID | Depth at train? | Depth at deploy? | Backbone modified? | Portable adapter? |
|---|---|---|---|---|---|
| DepthVLA | 2510.13375 | Yes (depth transformer) | **Yes (depth sensor required)** | Yes, full retrain | No |
| 3D-CAVLA | 2505.05800 | Yes (RGB-D point clouds) | **Yes (RGB-D required)** | Yes, full retrain | No |
| AugVLA-3D | 2602.10698 | Yes (VGGT depth) | **Yes (VGGT at inference)** | Yes (on GR00T only) | No |
| QDepth-VLA | 2510.14836 | Yes (ViDA pseudo-depth) | No — RGB only | Separate branch (not backbone-internal) | No (π0-specific) |
| MonoLift | NeurIPS 2025 | Yes (depth teacher) | No — RGB only | Policy-level distillation, not VLA backbone | Not demonstrated |
| UniLACT | 2602.20231 | Yes (RGB-D video) | No — RGB only | Latent action space, not vision encoder | No (GPT-2/ViT-L specific) |
| **DepthFree-VLA (ours)** | — | Yes (DAv2 pseudo-depth) | **No — RGB only** | **Depth-bottleneck adapter inside vision encoder** | **Yes — plug-in to any VLA** |

**The crux row is QDepth-VLA.** The residual delta is backbone-internal bottleneck architecture + cross-VLA portability. This is a real but narrow delta; reviewers will ask for direct comparison and may be skeptical without clear ablations.

---

## Method

**Overview.** A lightweight depth-bottleneck adapter (DBA) is inserted after the patch-embedding layer of a frozen VLA vision encoder (e.g., SigLIP-400M, ViT-L). During training, Depth Anything V2 (ViT-L teacher) generates dense pseudo-depth for every RGB frame in the demonstration dataset. The DBA learns to reconstruct depth tokens from its bottleneck representation as an auxiliary loss alongside the main action prediction objective. At deployment, the DBA runs on RGB alone — its internal representation has been shaped by depth reconstruction pressure, encoding geometric priors into the feature space without requiring a depth input.

**Architecture.**
- **Frozen VLA vision encoder** (e.g., SigLIP-400M) with patch embeddings producing tokens of dimension D.
- **Depth-bottleneck adapter (DBA):** a two-layer MLP that projects D → D/4 → D, with a side branch D/4 → N_depth that predicts N_depth depth tokens (discretized via a small VQ codebook or simply L2 regression against DAv2 outputs).
- **Auxiliary depth loss:** L_depth = MSE(predicted_depth_tokens, DAv2(RGB)) or cross-entropy over discretized depth bins. Weighted λ = 0.1 in the total loss.
- **Action loss:** standard VLA action prediction loss (cross-entropy or flow matching), unchanged.
- **Deployment:** only the DBA projection (D → D/4 → D path) runs; the depth-prediction side branch is dropped. Overhead: ~2–5 M adapter parameters, <1% of backbone.

**Depth Anything V2 teacher.** DAv2-ViT-L is run offline at dataset preparation time; depth maps are stored alongside RGB frames. No real-time depth model is needed at training or deployment. This is cheaper than ViDA (video-level temporal consistency) and simpler to integrate.

**Distillation target choice.** Unlike QDepth-VLA's VQ-VAE codebook (256 entries, dim 160), the DBA uses continuous L2 regression against normalised DAv2 outputs — simpler, avoids codebook training as a separate stage, and is compatible with any backbone dimension.

---

## Experimental Design

**Setup.** Fine-tune OpenVLA (7B, OXE-pretrained) and a compact RoboVLMs variant (3B) by inserting DBA into each model's vision encoder. Train on a 10K-episode OXE subset (Bridge v2 + FurnitureBench). Generate DAv2 pseudo-depth offline for all training frames.

**Evaluation protocol.**
- Remove pseudo-depth at test time (RGB-only rollout, depth branch dropped).
- Three robot platforms: tabletop (WidowX250), Franka arm (real), Google Robot (simulation via SimplerEnv).

**Primary metrics.**
| Metric | Rationale |
|---|---|
| Grasp success vs. object distance (20/40/60 cm) | Tests depth grounding on distance-stratified grasps |
| Thin/small-object grasp (pencil, cable tie) | Tests precision gained from spatial priors |
| Stacking precision (tower height 2/3 objects) | Tests relative depth ordering |
| Δ vs. vanilla VLA (no depth adapter) | Core ablation |
| Δ vs. QDepth-VLA (separate branch, same pseudo-depth) | Architecture comparison — critical for novelty |

**Baselines.**
1. Vanilla VLA (no depth signal at any stage).
2. QDepth-VLA-style replication (separate depth expert branch, dropped at deploy; same DAv2 pseudo-depth).
3. DepthVLA (2510.13375) — upper bound requiring depth sensor at deploy.
4. DBA adapter, depth branch kept active at inference (oracle upper bound for our method).

**The critical ablation** is Baseline 2 vs. DepthFree-VLA: if backbone-internal bottleneck does not outperform the separate-branch approach, the architectural novelty claim collapses.

---

## Why It's Publishable + Target Venue + Acceptance Hook

**Venue:** CoRL 2026 (primary) or RA-L with IROS 2026 (secondary).

**Acceptance hook.** The deployment cost argument is concrete: removing the depth camera cuts BOM cost by ~$200–$400 and eliminates a calibration failure mode. If the DBA matches or exceeds DepthVLA (which requires a sensor) on 2–3 benchmark tasks, the title "sensor-free depth grounding that matches sensor-based methods" is a strong hook. The plug-in framing (works with any frozen VLA) is relevant to the practitioner community.

**Risk to acceptance:** QDepth-VLA is the elephant in the room. Reviewers at CoRL will have read it. The paper must lead with the architectural distinction (backbone-internal bottleneck vs. separate branch) and include a direct ablation. Without that ablation, the paper is likely rejected for insufficient differentiation.

---

## Honest Win-Win Scorecard

| Claim | Verdict | Justification |
|---|---|---|
| Novel (unoccupied niche) | Partial ✓ | Core RGB-only deploy + training-time depth is scooped by QDepth-VLA; backbone-internal adapter framing is open |
| Better spatial precision | ? (needs verification) | QDepth-VLA shows +7–10% over vanilla; DBA must show > QDepth-VLA on at least one task |
| Less compute at deploy | ✓ | No depth sensor, no VGGT/DAv2 at inference, <5M adapter params |
| Faster training | ✗ | DAv2 pseudo-depth generation adds offline preprocessing; training is similar cost to QDepth-VLA |
| Cross-VLA portability | ✓ (claimed) | QDepth-VLA only shows π0; portability to OpenVLA is undemonstrated |
| Plug-in (no LLM retrain) | ✓ | Adapter-only; frozen backbone LLM |

---

## Risks & Kill Criteria

**#1 Risk — QDepth-VLA scoop (HIGH).** QDepth-VLA already achieves RGB-only deployment with training-time monocular depth pseudo-labels (ViDA, not DAv2) and shows +7–10 pp over vanilla on LIBERO and real-robot tasks. Any reviewer who has read QDepth-VLA will require a rigorous architectural ablation (separate branch vs. bottleneck) and a portability demonstration. If the DBA does not outperform the QDepth-VLA replication baseline by a statistically significant margin, the paper should not be submitted as-is.

**Kill criterion 1:** DBA does not improve over a separately-branched depth expert (QDepth-VLA replication) by ≥ 3 pp on the primary metric. Pivot: publish as a cross-architecture portability study — "how well does training-time depth distillation generalize across VLA families" — with QDepth-VLA as the baseline method applied to multiple backbones.

**Kill criterion 2:** DBA shows no gain over vanilla VLA (no depth) on any of the three evaluation settings. This would suggest the auxiliary depth signal is not propagating into action-relevant representations, indicating an architectural flaw in the bottleneck design.

**Kill criterion 3:** Latency or memory overhead of the DBA at inference is non-trivial (<1% claimed but must be verified on resource-constrained hardware).

**Secondary risk — MonoLift coverage.** MonoLift (NeurIPS 2025) establishes depth-to-RGB distillation as a legitimate direction; reviewers will ask why VLA-specific depth distillation is needed beyond MonoLift. The answer is language conditioning and generalist policy transfer, which MonoLift does not address.

**Secondary risk — UniLACT coverage.** UniLACT (2602.20231) also distills depth for RGB-only VLA inference. It differs architecturally (latent action space, not vision encoder) but a careful reviewer will ask for comparison. If UniLACT is available in public form by submission time, it should be a baseline.

---

## First De-risking Experiment (Week 1)

**Goal:** Determine whether the backbone-internal depth bottleneck produces meaningfully different visual representations than a separately-branched depth expert, before committing to full fine-tuning.

**Procedure:**
1. Take a pre-trained SigLIP-400M (frozen).
2. Train two variants on 1K BridgeV2 episodes with DAv2 pseudo-depth: (a) DBA (backbone-internal bottleneck), (b) separate depth head attached after the last layer (QDepth-VLA replication).
3. Extract patch token embeddings from both variants and compute linear probe accuracy on a simple object-distance classification task (3 bins: near/mid/far, labelled from DAv2 depth maps).
4. If DBA representations encode depth significantly better (≥5% linear probe Δ) than the separate-branch variant, the architectural hypothesis is supported. If no difference, the separate-branch design is sufficient and the DBA novelty claim is weak.

This experiment runs in < 1 GPU-day on a single A100 and gives a clear go/no-go signal for the full paper.

---

## Feasibility

**Medium.** The components are available: DAv2 is publicly released, OXE is accessible, OpenVLA checkpoints are public. The main uncertainties are (1) whether the bottleneck design actually produces distinct representations from QDepth-VLA's approach (requires Week-1 experiment), and (2) whether CoRL reviewers accept the architectural distinction as sufficient novelty given QDepth-VLA's prior claim. Timeline: 6–8 weeks for full implementation and 3-platform evaluation.

---

## Key References

| Title | Venue / Year | arXiv ID |
|---|---|---|
| QDepth-VLA: Quantized Depth Prediction as Auxiliary Supervision for Vision-Language-Action Models | arXiv Oct 2025 | 2510.14836 |
| DepthVLA: Enhancing Vision-Language-Action Models with Depth-Aware Spatial Reasoning | arXiv Oct 2025 | 2510.13375 |
| 10 Open Challenges Steering the Future of Vision-Language-Action Models | arXiv Nov 2025 | 2511.05936 |
| 3D-CAVLA: 3D Context-Aware Vision-Language-Action Model | arXiv May 2025 | 2505.05800 |
| Depth Anything V2 | NeurIPS 2024 | 2406.09414 |
| MonoLift: Learning 3D Manipulation Policies from Monocular RGB via Distillation | NeurIPS 2025 | OpenReview wZzC5rpDY1 |
| UniLACT: Depth-Aware RGB Latent Action Learning for Vision-Language-Action Models | arXiv Feb 2026 | 2602.20231 |
| AugVLA-3D: Depth-Driven Feature Augmentation for Vision-Language-Action Models | arXiv Feb 2026 | 2602.10698 |
| OpenVLA: An Open-Source Vision-Language-Action Model | arXiv Jun 2024 | 2406.09246 |

---

*Scoop check performed 2026-05-28. QDepth-VLA (2510.14836) is the primary overlap; its RGB-only deploy claim is confirmed. The defensible DepthFree-VLA delta — backbone-internal bottleneck adapter, cross-VLA portability, Depth Anything V2 as teacher — is narrow and requires the Week-1 de-risking experiment before committing to a full paper.*


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
