# Project 4 — OXE-Scale Coreset & Selection Scaling-Law

## TL;DR (3-4 lines)
Use **cheap, scalable proxy scores** (EL2N, GraNd/gradient-norm, loss-trajectory variance, embedding-diversity) — *not* expensive influence functions or datamodels — to select an Open X-Embodiment (OXE) coreset that matches the full-mixture generalist VLA at a fraction of the data and pretraining compute. The defensible contribution is a **selection scaling-law at generalist-pretraining scale** (performance vs. retained-data fraction, fit per proxy and extrapolated), plus the empirical finding that **pruning negative-transfer trajectories can match or beat the full mix**. This is a data-centric win-win (less data + less compute), but it is honestly **scoop-adjacent** and earns its keep on the *cheap-proxy + pretraining-scale + scaling-law* combination, not on selection alone.

## The gap / motivation
Robot data is the bottleneck, and the field already knows curation works — but *only at single-task fine-tuning scale*. CUPID (influence functions, <33% data → SOTA on RoboMimic), QoQ (2603.09056), DataMIL (datamodels), and MI-estimator curation (2502.08623) all curate a small dataset to improve **one downstream task**. Separately, 2602.09722 shows that naively pooling OXE induces **negative transfer**, but treats this as a *mixture-weighting / representation-alignment* problem at the dataset/embodiment granularity. Nobody has asked the *generalist-pretraining* question: **can a cheap, per-trajectory proxy prune OXE to a coreset that reproduces the full-mixture generalist VLA, and does the resulting performance-vs-data curve obey a predictable scaling-law?** Influence functions and datamodels do not scale to ~1M OXE trajectories without heroic compute; cheap proxies do. That scalability gap is the opening.

## Precise novelty & positioning

| Paper | arXiv id | Scale & method | Our delta |
|---|---|---|---|
| CUPID | 2506.19121 | Fine-tuning scale; **influence functions** on closed-loop return; single-task / post-training | We prune at **OXE generalist-pretraining** scale with **cheap proxies** (no Hessian/influence); we fit a **scaling-law**, not a single subset |
| Rethinking VLA Scaling (CLOSEST) | 2602.09722 | OXE **mixture weighting + EEF-relative alignment**; dataset/embodiment granularity; no scaling-law | We do **per-trajectory coreset selection** via cheap scores, **orthogonal** to their mixture/representation knobs; we produce an extrapolatable **selection scaling-law** |
| FT-NCFM | 2511.16233 | Benchmark scale; influence-aware **data distillation** (synthesizes a 5% coreset) | We **select real trajectories** (no synthesis), at OXE scale, with a scaling-law and negative-transfer analysis |
| DataMIL | 2505.09603 | Selects OXE data **for a single downstream task** via **datamodels** (many proxy runs) | We build a **task-agnostic generalist** coreset with **cheaper** scores and report a scaling-law; not task-conditioned |
| QoQ | 2603.09056 | Fine-tuning; **influence functions** for demo quality | Same delta as CUPID: cheap proxy + pretraining scale + scaling-law |
| Robot Data Curation w/ MI | 2502.08623 | Per-dataset; **MI / kNN-VAE** quality scoring | We benchmark MI as one proxy; our object is the generalist scaling-law, not per-demo quality |
| RoboVLMs | 2412.14058 | VLA **design** study (backbones/architectures) | We supply the VLA-class testbed; our axis is *data selection*, not design |

**Crux:** the delta is the *intersection* — cheap-proxy AND pretraining-scale AND a fitted selection scaling-law. Any single axis alone is arguably covered.

## Method
**Cheap proxy scorers** (all O(1–2 forward/backward passes per trajectory, no Hessian, no proxy-model zoo): (i) **EL2N** — error-L2-norm of action prediction early in a short reference run; (ii) **GraNd** — per-trajectory gradient-norm; (iii) **loss-trajectory** — mean/variance of action loss across a few checkpoints of one cheap reference VLA; (iv) **embedding-diversity** — facility-location / kNN coverage over a frozen vision-language embedding to retain diverse, drop redundant trajectories. Optionally combine difficulty+diversity (D2-style) to avoid keeping only hard outliers.
**Selection pipeline:** train one short, small **reference** VLA on a uniform OXE slice → compute proxy scores for all trajectories → keep top/coverage fraction *f* (with per-embodiment quotas to avoid collapsing diversity) → **retrain from scratch** at *f* ∈ {30, 50, 70, 100}%.
**Scaling-law fitting:** fit `Perf(f) = a − b·f^(−c)` (and a saturating logistic variant) per proxy across fractions and ≥2 model sizes; report extrapolation error and the **break-even fraction** where the coreset matches full-mix. Negative-transfer test: explicitly *remove* the lowest-scoring trajectories and check whether ≤100% beats 100%.

## Experimental design
- **Subsample OXE** to 30/50/70% via each proxy (+ per-embodiment quotas).
- **Retrain** a small **RoboVLM-class** VLA (~1–3B; e.g. a flow/diffusion-action head on a compact VLM) from scratch at each fraction; ≥2 sizes for scaling-law extrapolation.
- **Eval**: **LIBERO** (spatial/object/goal/long) and **SimplerEnv** (Bridge/Fractal); report success-rate vs. retained-data and vs. training-FLOPs.
- **Baselines**: (1) **random** subset, (2) **uniform-mixture** / OXE default weights, and (if affordable) (3) one **influence-function / datamodel** point at 50% to show cheap proxies recover most of the gain at a fraction of the selection cost.
- **Deliverable**: a per-proxy selection scaling-law + the released coreset indices.

## Why it's publishable + target venue + acceptance hook
Target: **NeurIPS / ICLR Datasets & Benchmarks** (a reusable OXE coreset + selection scaling-law is a clean D&B artifact) or **CoRL**. Hook: the first *selection* scaling-law for generalist robot pretraining — a predictive tool ("at fraction *f*, expect performance *p*; break-even at *f\**") plus a released coreset and scorer code that the community can reuse and extend.

## Honest win-win scorecard
- **Novel?** ⚠️ **Partial.** Selection-for-robots is published (CUPID/QoQ/DataMIL); mixture/negative-transfer is published (2602.09722). Novelty is the *cheap-proxy + pretraining-scale + scaling-law* combination, not selection per se.
- **Better?** ⚠️ Plausible only via the negative-transfer effect (≤100% ≥ 100%); otherwise the honest bar is *match* full-mix, not beat it.
- **Less data?** ✓ Core claim (30–70% of OXE).
- **Less compute?** ✓ Both selection (cheap proxies vs. influence/datamodels) and training (smaller mixture).
- **Faster?** ✓ At inference no change, but training/iteration is faster; selection itself is cheap.

## Risks, scoop-adjacency & kill-criteria
**Strongest objection:** *"Isn't this just 2602.09722 / CUPID at bigger scale?"* — Honest answer: **partly, and this is the weakest/most scoop-exposed proposal in the set.** Concession: if a reviewer reads "OXE data selection" as already-solved by DataMIL + 2602.09722, the delta is thin. Defense: (a) DataMIL is *task-conditioned* and uses *datamodels* (many proxy runs); we are *task-agnostic generalist* with *single-pass* proxies — a real cost/scope difference. (b) 2602.09722 never does per-trajectory selection and never fits a scaling-law; our axis is orthogonal and *composes* with theirs. (c) No prior work reports a **selection** scaling-law for robot pretraining (existing robot scaling laws — 2410.18647, 2505.07728 — scale *collection*, not *pruned fraction*). **Kill-criteria:** (1) if cheap proxies do not beat *random* by a clear margin at OXE scale (the data-diet result may not transfer from vision to multi-embodiment robot data); (2) if the scaling-law does not extrapolate across model sizes (then it is not a "law"); (3) if a concurrent paper publishes cheap-proxy generalist OXE coresets → flip to a focused *negative-transfer pruning* study or a *proxy-benchmark* contribution.

## First de-risking experiment (week 1)
**One proxy (EL2N), 50% OXE, one small VLA, vs. random-50% and full-100%, LIBERO only.** Success = EL2N-50% ≥ random-50% by a clear margin **and** within a small gap of full-100%. If EL2N-50% ≈ random-50%, cheap proxies do not transfer to OXE-scale robot data → kill or pivot before spending on the full grid.

## Feasibility
**Compute-HEAVIER than the study-style proposals**: requires *retraining* small VLAs from scratch at multiple fractions × ≥2 sizes (the scaling-law needs several points). Proxy scoring is cheap; the cost is the retrain grid. Mitigations: use a small (~1–3B) RoboVLM-class model, restrict to LIBERO + SimplerEnv (no hardware), and reuse the single reference run for scoring. Influence-function baseline is optional/single-point precisely because it does *not* scale — which is the paper's point.

## Key references
- **CUPID: Curating Data your Robot Loves with Influence Functions** — CoRL 2025, arXiv:2506.19121
- **Rethinking Visual-Language-Action Model Scaling: Alignment, Mixture, and Regularization** — 2026, arXiv:2602.09722
- **FT-NCFM: An Influence-Aware Data Distillation Framework for Efficient VLA Models** — 2025, arXiv:2511.16233
- **DataMIL: Selecting Data for Robot Imitation Learning with Datamodels** — 2025, arXiv:2505.09603
- **Quality over Quantity: Demonstration Curation via Influence Functions** — 2026, arXiv:2603.09056
- **Robot Data Curation with Mutual Information Estimators** — 2025, arXiv:2502.08623
- **What Matters in Building Vision-Language-Action Models (RoboVLMs)** — Nature Machine Intelligence 2025, arXiv:2412.14058
- **Deep Learning on a Data Diet (EL2N/GraNd)** — NeurIPS 2021, arXiv:2107.07075
- **Data Scaling Laws in Imitation Learning for Robotic Manipulation** — ICLR 2025, arXiv:2410.18647
- **Guiding Data Collection via Factored Scaling Curves** — 2025, arXiv:2505.07728


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
