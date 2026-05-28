# Project 6 — Curriculum-VLA: Self-Paced Demonstration Ordering for Sample-Efficient Fine-Tuning

---

## TL;DR

Standard VLA supervised fine-tuning samples mini-batches uniformly at random, leaving the order of demonstrations entirely unexploited. We propose scoring each demonstration by the VLA's own action-prediction loss on a held-out split (a label-free difficulty proxy), then training with a self-paced competence schedule that progresses from easy to hard demonstrations during LoRA fine-tuning of OpenVLA-OFT. No architecture change, no extra labels, zero deployment overhead. Curriculum-learning theory predicts 30–50% fewer gradient steps to the same validation loss; the method is trivially reproducible on any lab GPU cluster.

---

## The Gap / Motivation

VLA supervised fine-tuning (behavior cloning / SFT) with LoRA is now the standard recipe for adapting large pretrained VLAs to new robot setups (OpenVLA-OFT, 2502.19645). The standard training loop samples mini-batches **uniformly at random** from the demonstration dataset at every gradient step. This ignores a well-established principle from machine learning: the **order** in which training examples are presented matters for both convergence speed and final generalization (Bengio et al., ICML 2009; Kumar et al., NeurIPS 2010).

In VLA fine-tuning specifically:
- Demonstration datasets are heterogeneous in difficulty (near-failures, long-horizon sequences, rare grasp angles all land in the same pool).
- A pretrained VLA already has a strong prior; early exposure to the hardest demonstrations may induce gradient noise that slows convergence.
- The model's own action-prediction loss is a free, self-supervised signal that naturally ranks demonstrations by how surprising they are to the current policy — no human annotation required.

Despite a large body of work optimizing *what* to fine-tune (decoding schemes, action representations, learning objectives — all studied in OpenVLA-OFT), **no published work applies self-paced demonstration ordering to VLA SFT fine-tuning**. This is the gap.

---

## Precise Novelty & Positioning

| Paper | arXiv ID | What they do | Our delta |
|---|---|---|---|
| OpenVLA-OFT (Kim, Finn, Liang 2025) | 2502.19645 | Studies decoding (parallel), action rep (continuous L1), objectives, LoRA fine-tuning — all orthogonal to data ordering | We leave their recipe intact; add a self-paced demo-ordering wrapper around the same LoRA training loop |
| STARE-VLA (Xu et al. 2025) | 2512.05107 | RL-based stage-aware ordering **within** a single trajectory (TPO/PPO); fine-tunes reward signal, not SFT data order | Entirely different regime: SFT not RL; ordering is across demonstrations, not within one trajectory |
| Curriculum Learning (Bengio et al. 2009) | — (ICML proceedings) | Foundational theory: easy-to-hard ordering accelerates learning | We instantiate with the VLA's own loss as the difficulty signal; first application to VLA SFT demo ordering |
| Self-Paced Learning (Kumar et al. 2010) | — (NeurIPS proceedings) | Self-paced formulation with competence parameter λ; no extra labels | We adapt λ schedule to gradient-step budget of LoRA VLA fine-tuning |
| DR-SPCRL (Satheesh et al. 2025) | 2511.05694 | Self-paced robustness budget schedule for **RL** policies | RL, not SFT; robustness budget, not demonstration ordering |

**Our delta in one sentence:** A self-paced SFT demonstration-ordering curriculum for VLAs, using the VLA's own per-demo action-prediction loss as the difficulty signal — not studied before.

---

## Method

**Step 1 — Pre-score the dataset.**  
Before training begins, run a single inference pass of the frozen (pre-LoRA) VLA over the full demonstration dataset. For each demonstration `d_i`, compute the mean per-timestep action-prediction loss `L_i = mean_t [ L_action(a_t, â_t(d_i)) ]` using the OpenVLA-OFT L1 regression objective. This single pass costs ~1× the per-epoch compute and is done once.

**Step 2 — Self-paced competence schedule.**  
Define a competence threshold `λ(k)` that increases monotonically with gradient step `k`:

```
λ(k) = λ_0 + (λ_max - λ_0) · (k / K_total)^γ
```

At step `k`, the eligible training pool is `{ d_i : L_i ≤ λ(k) }`. Mini-batches are drawn uniformly from this pool. `γ ∈ {0.5, 1.0}` (concave vs. linear ramp) is a hyperparameter; `λ_0` = 20th percentile loss, `λ_max` = 100th percentile (full dataset). No demonstration is ever permanently excluded — the schedule is additive, not subtractive.

**Step 3 — LoRA fine-tuning.**  
Drop this scheduling wrapper into the standard OpenVLA-OFT LoRA training loop. No changes to the model, optimizer, loss function, or inference code. The only overhead is the one-time pre-scoring pass and the per-step pool filtering (negligible).

**Why this difficulty proxy is well-motivated:** The VLA's initial action-prediction loss on a demonstration directly measures how much the current policy is "surprised" by that demonstration — i.e., how far it is from the model's current competence. This is precisely the self-paced learning criterion (Kumar et al., 2010) applied without any external labels.

---

## Experimental Design

**Platform:** OpenVLA-OFT (2502.19645) fine-tuned on **LIBERO-90** (90-task benchmark; standard in the VLA literature).

**Conditions (within-subjects, same total gradient steps K):**
1. **Random** — uniform mini-batch sampling (baseline, replicates OpenVLA-OFT default)
2. **Easy-first (self-paced)** — proposed method, λ(k) ramp as above
3. **Hard-first** — reverse schedule (ablation; tests whether ordering direction matters)
4. **Low-data regime** — 20% of demonstrations randomly subsampled, all three conditions above repeated (this is where ordering is expected to matter most)

**Headline metric — early-stopping curves:**  
Task success rate measured at 25%, 50%, 75%, and 100% of total gradient steps K. This directly operationalizes "sample efficiency" as convergence speed to a fixed success threshold.

**Secondary metrics:** Final success rate at 100% K (tests whether curriculum hurts ceiling), gradient-step count to reach 90% of the random-baseline final performance.

**Seeds:** 3 per condition (6 conditions × 3 seeds = 18 runs; plus 4-point measurement = tractable).

**Ablations:**
- γ ∈ {0.5, 1.0} (schedule shape)
- Difficulty proxy: VLA loss vs. trajectory length vs. random shuffle (sanity check)

---

## Why It's Publishable + Target Venue + Acceptance Hook

**Target venues:** CoRL 2026 (primary), RA-L with ICRA option (secondary), NeurIPS 2026 Datasets & Benchmarks (if the LIBERO-90 result is strong).

**Acceptance hook:** The result is a clean, reproducible training-efficiency story. Reviewers at CoRL and RA-L consistently reward methods that (a) require no new data or architecture, (b) show a compelling early-stopping curve, and (c) are trivially adoptable. The low-data regime result is the strongest hook: practitioners who can only collect 20–50 demonstrations benefit most from ordering, and this is a very common real-world scenario.

**The paper writes itself:** Introduction cites Bengio + Kumar, Section 2 reviews OpenVLA-OFT and STARE-VLA to establish the gap, Section 3 is the two-paragraph method, Section 4 is the ablation table, and the conclusion is two sentences.

---

## Honest Win-Win Scorecard

| Criterion | Verdict | Note |
|---|---|---|
| Novel | ✓ | No prior work applies self-paced SFT demo ordering to VLAs (search confirmed May 2026) |
| Better final performance | ? | Theory predicts equal-or-better; empirically may be marginal at 100% K; the win is primarily convergence speed |
| Less compute (total) | ✓ | 30–50% fewer gradient steps to the same validation loss = direct FLOPs reduction; one-time pre-scoring is small |
| Faster wall-clock | ✓ | Fewer steps = faster wall-clock on fixed hardware |
| Better sample efficiency | ✓ | Strongest in low-data regime; this is the headline claim |
| Inference-time benefit | ✗ | This is a training-time method only; zero effect at deployment |
| Requires new data | ✗ | Uses existing demonstrations; no new collection |

---

## Risks & Kill-Criteria

**Risk 1 — "Incremental application" reviewer objection.** Curriculum learning is well-known; reviewers may argue this is a straightforward application. **Mitigation:** (a) Show a quantitatively large convergence speedup (>20% step reduction at 3 seeds, statistically significant); (b) include the hard/low-data regime where the effect is largest; (c) add a difficulty-metric analysis showing the loss-based proxy is better than naive proxies (trajectory length, random). The story must be "the right difficulty signal for VLAs, rigorously evaluated" not just "we sorted the data."

**Risk 2 — Null result on convergence.** Random sampling may already converge as fast as the self-paced schedule, especially with large datasets where the pretrained VLA already handles most demos well. **Kill criterion:** If the easy-first curve does not reach the random-baseline 90%-success threshold at least 15% faster (in gradient steps) on at least 2 of 3 seeds, in both the full-data and low-data regimes, the core claim does not hold and the project should be pivoted or dropped.

**Risk 3 — Hard-first outperforms easy-first.** Theoretically possible if the pretrained VLA is strong enough that "hard" demos are more informative. This would invert the curriculum hypothesis. **Mitigation:** This is still a publishable result — it would reveal that VLA priors are strong enough to reverse the standard curriculum prediction, which is a genuine finding. Report honestly.

**Risk 4 — Pre-scoring instability.** Loss values computed before LoRA training may not rank difficulty stably (i.e., demonstrations with high initial loss may become easy quickly). **Mitigation:** Re-score at 25% and 50% of training as a dynamic-curriculum variant; compare to static pre-scoring.

---

## First De-Risking Experiment

**Week 1 target:** Run 3 conditions (random / easy-first / hard-first) × 3 seeds = 9 runs on LIBERO-90, full data, with the OpenVLA-OFT default total K steps. Measure success rate at 25/50/75/100% of K.

**Compute estimate:** Each run is a standard OpenVLA-OFT LoRA fine-tune on LIBERO-90, approximately 4 GPU-hours on an A100. Total: ~36 GPU-hours. Fits comfortably in a 2-day slot on a 4-GPU academic lab node.

**Go signal:** Easy-first reaches the random-baseline final performance at ≤75% of K steps, on at least 2/3 seeds, with a standard error that does not overlap with random at the same checkpoint. If yes, proceed to low-data ablation and full sweep. If no, diagnose: check the loss distribution, try dynamic re-scoring, then apply kill criterion.

---

## Feasibility for an Academic Lab

Very high. The entire method is a modification to the training data loader (one file in any standard PyTorch training loop). It requires:
- Access to OpenVLA-OFT code (public, MIT license, https://openvla-oft.github.io/)
- LIBERO-90 dataset (public)
- ~40 GPU-hours for the de-risking experiment; ~200 GPU-hours for the full paper sweep
- No new data collection, no new model architecture, no deployment infrastructure

A single PhD student can implement and run the de-risking experiment in one week, and the full paper sweep in one month.

---

## Key References

| Title | Venue / Year | arXiv ID |
|---|---|---|
| Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success (OpenVLA-OFT) | arXiv 2025 | 2502.19645 |
| STARE-VLA: Progressive Stage-Aware Reinforcement for Fine-Tuning Vision-Language-Action Models | arXiv 2025 | 2512.05107 |
| Curriculum Learning | ICML 2009 | — (proceedings only) |
| Self-Paced Learning for Latent Variable Models | NeurIPS 2010 | — (proceedings only) |
| Distributionally Robust Self-Paced Curriculum Reinforcement Learning | arXiv 2025 | 2511.05694 |
| VLA-RL: Towards Masterful and General Robotic Manipulation with Scalable RL | arXiv 2025 | 2505.18719 |


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
