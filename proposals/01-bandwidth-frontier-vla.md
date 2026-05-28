# Project 1 — Bandwidth-Frontier VLA

## TL;DR (3-4 lines)
A language-conditioned VLA (OpenVLA-OFT or π0) emits a **low-rate (5–10 Hz) force/impedance reference** (target wrench + impedance gains), and a **tiny analytic admittance + ≤1-layer learned residual** controller closes the loop at **≥200 Hz (up to 1000 Hz)** using only proprioception + 6-axis F/T — **no vision in the fast loop**. The headline is a **control-bandwidth / contact-stability frontier**: contact-force overshoot, disturbance-rejection bandwidth, and success vs. inner-loop rate. Verified May 2026: no VLA paper reports this evaluation axis, so we change the axis rather than fight the saturated "another fast-slow box" race.

## The gap / motivation
Contact stability is fundamentally a **bandwidth** problem (Colgate–Hogan passivity): closing a force loop at 9–60 Hz cannot reject contact transients or guarantee passivity against stiff environments. Every recent force-aware VLA still closes its reactive loop with a **neural network** at 9–60 Hz (TIDAL ~9 Hz, AT-VLA ~25 Hz, DuoCore-FS 30 Hz, M2-ResiPolicy 60 Hz) and reports only success/peak-force. None reports the curve every controls engineer would ask for: **how does contact behavior degrade as the inner loop slows from 1000 → 15 Hz, and where is the knee?** That curve is the contribution.

## Precise novelty & positioning

| Paper | arXiv id | What they do | Our delta |
|---|---|---|---|
| OpenVLA-OFT | 2502.19645 | Optimized VLA fine-tuning (parallel decode, L1 chunks); base model | We use it as the **reference head**, not the controller |
| π0 | 2410.24164 | Flow-matching generalist VLA; base model | Alternate reference head; we add an analytic fast loop |
| FAVLA | 2602.23648 | Force-adaptive fast–slow VLA; **fully neural** action expert at *variable* high rate | We use **analytic admittance + thin residual** (passivity-shapeable) and report a **bandwidth frontier** + fixed-rate sweep; they report neither |
| M2-ResiPolicy | 2603.15152 | 60 Hz GRU residual on TCP wrench | **Non-VLA base**; we are language-conditioned and run ≥200 Hz with an analytic core |
| TIDAL | 2601.14945 | Interleaved diffusion+action, ~9 Hz, **vision/proprio only** | We add 6-axis force and a ≥200 Hz analytic loop |
| DuoCore-FS | 2512.20188 | Async fast–slow VLA, 30 Hz whole-body | No force inner loop, no analytic core, 30 Hz ≪ 200 Hz |
| AT-VLA | 2605.07308 | Adaptive tactile injection, ~25 Hz dual-stream | Tactile not 6-axis wrench; 25 Hz; no analytic loop/frontier |
| ForceVLA / ForceVLA2 | 2505.22159 / 2603.15169 | Force-aware MoE / hybrid force-position VLA | End-to-end; no high-rate analytic loop, no rate sweep/frontier |
| RTC | 2506.07339 | Real-time chunking (latency hiding) | Orthogonal; hides inference latency, not a control loop — **composable** with us |
| Reactive Diffusion Policy | 2503.02881 | Slow-fast visual-tactile; fast tokenizer decoder | Decoder, not analytic controller; no passivity/frontier reporting |
| CompliantVLA-adaptor | 2601.15541 | VLM-set variable impedance params + F/T clamp | Sets gains but **no high-rate learned residual**, no frontier |
| TA-VLA / OmniVIC | 2509.07962 / 2510.17150 | Torque-aware VLA / VLM-guided VIC | Inputs/safety, not a ≥200 Hz analytic+residual loop with a frontier result |

**Delta in one line:** (a) couple a *language-conditioned VLA* to a *≥200 Hz analytic-admittance + thin-residual* inner loop; (b) report the *control-bandwidth frontier* nobody reports.

## Method
**Reference head (5–10 Hz).** OpenVLA-OFT (or π0) consumes RGB + language + proprioception and emits a structured reference at chunk rate: target end-effector pose/wrench `(x_d, F_d)` and **impedance parameters** `(K, D)` (diagonal 6-DoF), plus a confidence/contact-phase token. The head never sees F/T at speed; it sets *intent*.

**Inner loop (≥200 Hz).** Standard admittance law `M ẍ + D(ẋ − ẋ_d) + K(x − x_d) = F_d − F_ext`, integrated to a velocity/position command and tracked by the robot's torque/joint controller. F_ext comes from the wrist F/T sensor; everything is proprioceptive. On top sits a **≤1-layer learned residual** `δ = MLP(F_ext, ẋ, e, K, D)` (one hidden layer or a 1-step GRU, <5k params) that corrects model-mismatch (sensor lag, unmodeled friction) — small enough to keep the loop analytically dominant and **passivity-monitorable** (energy-tank / passivity observer gates the residual; it is clipped if it injects energy).

**How the two rates interface.** The reference `(x_d, F_d, K, D)` is **zero-order-held** and rate-limited between VLA updates; RTC-style chunk overlap can hide head latency. The residual is trained with the head frozen, so the slow policy is reusable as-is — the contribution is the inner loop, not a new VLA.

**Training.** (1) Pretrain/fine-tune the VLA head on contact tasks for `(x_d, F_d, K, D)` via behavior cloning (force labels from sim F/T or demo wrench). (2) Train the residual by imitation + a small contact-shaping loss (penalize overshoot beyond `F_d`, reward tracking) under domain randomization over environment stiffness, sensor delay, and F/T noise. Curriculum on inner-loop rate so the residual is rate-robust.

## Experimental design
**Models.** Reference head ∈ {OpenVLA-OFT, π0}. Inner loop ∈ {analytic-only, analytic+residual}.

**Sim (primary).** ManiSkill / SimplerEnv contact-rich tasks (peg-in-hole, surface wiping, plug insertion, door/drawer with stiff stops). Simulate F/T at the physics rate and **decimate to a controllable inner-loop rate (15 → 1000 Hz)**; inject sensor delay + noise to mimic a real wrist F/T.

**Baselines.** Open-loop VLA chunks; VLA + neural fast loop reproductions at their native rates (FAVLA-style variable-rate AE, M2-style 60 Hz GRU, RDP-style fast decoder); CompliantVLA-style VLM-set impedance with no residual.

**Metrics (the new axis).** Peak/steady **contact-force overshoot** (N, % over `F_d`); **disturbance-rejection bandwidth** (−3 dB from injected force perturbations); **passivity margin** (energy-tank violations); task success. Report all *as a function of inner-loop rate*.

**The frontier (headline figure).** **Inner-loop rate sweep 15 → 1000 Hz** for analytic, analytic+residual, and neural baselines, plotting overshoot ↓ / bandwidth ↑ / success ↑ vs. rate and locating the **knee** (expected ~100–200 Hz for analytic; neural baselines plateau lower and saturate by their native rate).

**Ablations.** Analytic-only vs. +residual; residual size (0 → 1 → 2 layers); with/without passivity gate; ZOH vs. rate-limited reference; head update rate 5 vs. 10 Hz; environment-stiffness generalization.

## Why it's publishable + target venue + acceptance hook
**Venue:** CoRL or RA-L/ICRA. **Hook:** the first **control-bandwidth / contact-stability frontier for VLAs** — a reproducible sim protocol + curve that reframes "fast-slow VLA" as a *bandwidth* question and gives the community a missing evaluation axis. Even if absolute success ties the neural fast-slow SOTA, the **frontier characterization + passivity guarantees + tiny analytic loop** are the accepted contribution. Composability with RTC and reuse of frozen heads make it immediately adoptable.

## Honest win-win scorecard
- **Novel:** ✓ — analytic+residual ≥200 Hz loop on a language VLA *and* a bandwidth-frontier axis no VLA paper reports (verified May 2026).
- **Better:** ? — expect lower force-overshoot / higher disturbance-rejection bandwidth than 9–60 Hz neural loops; task-success gain is a hypothesis to be earned, not assumed.
- **Less-compute:** ✓ — inner loop is analytic + <5k-param residual; far cheaper than a transformer action expert at speed; frozen reusable head.
- **Faster:** ✓ — ≥200 Hz (to 1000 Hz) reactive loop vs. competitors' 9–60 Hz, with low compute per step.

## Risks, scoop-fuse, kill-criteria
- **Scoop cadence:** fast-slow VLA is moving ~weekly (FAVLA, ForceVLA2, M2-ResiPolicy all Feb–Mar 2026). **Re-run the 12-query verification every ~3 weeks**; specific fuse — *a paper that (i) puts an analytic or learned inner loop ≥100 Hz on a language-conditioned VLA AND (ii) reports a rate sweep / disturbance-rejection frontier* triggers a pivot (toward passivity-certified guarantees or real-hardware stiffness-generalization, which abstracts above pure rate).
- **Kill-criteria (null result):** if the frontier shows **no knee** — i.e., success and overshoot are flat from 15 → 1000 Hz on realistic tasks — the premise is dead and we report it as a negative result (still informative: "bandwidth doesn't matter for these tasks"). Also kill if the analytic+residual loop cannot beat a 60 Hz neural loop on *any* metric.
- **Sim-real gap:** sim F/T is idealized; the frontier knee may shift on hardware. Mitigate with delay/noise randomization and an optional real-robot spot-check.

## First de-risking experiment (week-1 plan)
Single ManiSkill peg-in-hole task; frozen OpenVLA-OFT head emitting `(x_d, F_d, K, D)` at 5 Hz (or scripted reference if head not yet tuned). Implement **analytic-only** admittance and run the **rate sweep 15 → 1000 Hz**, plotting force-overshoot and success. **Go/no-go:** a visible knee (overshoot rising sharply below ~100–200 Hz) confirms the frontier exists *before* any residual/learning is built.

## Feasibility for an academic lab
**Sim-first, single-GPU.** The novelty lives in the inner loop + evaluation protocol, not in training a new foundation model — use *released* OpenVLA-OFT/π0 checkpoints, fine-tune the small reference head and the <5k-param residual on one GPU. Optional real-robot validation: any **7-DoF arm + wrist 6-axis F/T** (e.g., Franka + ATI/Bota) running the analytic loop at 500–1000 Hz; the slow head can run on the same GPU at 5–10 Hz.

## Key references
- *Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success* (OpenVLA-OFT), 2025 — 2502.19645
- *π0: A Vision-Language-Action Flow Model for General Robot Control*, 2024 — 2410.24164
- *FAVLA: A Force-Adaptive Fast–Slow VLA Model for Contact-Rich Manipulation*, 2026 — 2602.23648
- *M2-ResiPolicy: Micro Residual Correction with Adaptive Tactile Fusion and Force-Mixed Control*, 2026 — 2603.15152
- *TIDAL: Temporally Interleaved Diffusion and Action Loop for High-Frequency VLA Control*, 2026 — 2601.14945
- *DuoCore-FS: Asynchronous Fast–Slow VLA Policies for Whole-Body Manipulation*, 2025 — 2512.20188
- *AT-VLA: Adaptive Tactile Injection for Enhanced Feedback Reaction in VLA Models*, 2026 — 2605.07308
- *ForceVLA: Enhancing VLA with a Force-aware MoE for Contact-Rich Manipulation*, 2025 — 2505.22159
- *ForceVLA2: Hybrid Force-Position Control with Force Awareness*, 2026 — 2603.15169
- *Real-Time Execution of Action Chunking Flow Policies* (RTC), 2025 — 2506.07339
- *Reactive Diffusion Policy: Slow-Fast Visual-Tactile Policy Learning*, 2025 — 2503.02881
- *CompliantVLA-adaptor: VLM-Guided Variable Impedance Action*, 2026 — 2601.15541
- *TA-VLA: Elucidating the Design Space of Torque-aware VLA Models*, 2025 — 2509.07962
- *OmniVIC: Self-Improving Variable Impedance Controller with Vision-Language In-Context Learning*, 2025 — 2510.17150
- *Admittance Visuomotor Policy Learning for Contact-Rich Manipulation*, 2024 — 2409.14440


---

*Part of the [VLA Research Proposal Portfolio](../README.md) · Methodology: [docs/methodology.md](../docs/methodology.md) · Kill-log: [docs/research-log.md](../docs/research-log.md)*
