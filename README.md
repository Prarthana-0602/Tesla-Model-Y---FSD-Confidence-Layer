<div align="center">

<h1>⚡ FSD Confidence Layer</h1>

<p><strong>Tesla Model Y · Full Self-Driving (Supervised)</strong></p>

<p><em>A software-only OTA feature that shows drivers what FSD is about to do, and how confident it is,<br/>1–3 seconds before every high-risk maneuver.</em></p>

<br/>

[![Prototype](https://img.shields.io/badge/Prototype-Live-brightgreen?style=for-the-badge)](https://preeminent-sherbet-e3685d.netlify.app/)
[![Demo](https://img.shields.io/badge/Loom-Watch%20Demo-blue?style=for-the-badge)](https://www.loom.com/share/233a8c3e993e4dc69fec0993fe3004f3)
[![Research](https://img.shields.io/badge/User%20Research-12%20Participants-orange?style=for-the-badge)](https://forms.gle/CjJcSuY6gaznYfnT6)
[![Course](https://img.shields.io/badge/UCLA%20Anderson-MGMT%20275-darkblue?style=for-the-badge)](https://www.anderson.ucla.edu/)

<br/>

**[View Prototype](https://preeminent-sherbet-e3685d.netlify.app/) · [Watch Demo](https://www.loom.com/share/233a8c3e993e4dc69fec0993fe3004f3) · [User Interview Form](https://forms.gle/CjJcSuY6gaznYfnT6)**

</div>

---

> *"I keep my hand near the wheel the whole time because I never know when it's going to hesitate at an intersection."*
> — Active FSD Subscriber, Los Angeles Metro

---

## 📌 Table of Contents

- [About the Project](#-about-the-project)
- [The Two Layers](#-the-two-layers)
- [Scenarios in Scope](#-scenarios-in-scope-mvp)
- [How to Use the Prototype](#-how-to-use-the-prototype)
- [Market Research & Validation](#-market-research--validation)
- [Product Strategy & Prioritization](#-product-strategy--prioritization)
- [Success Metrics & Go-To-Market](#-success-metrics--go-to-market)
- [Team](#-team)
- [Sources](#-sources)

---

## 💡 About the Project

### 🎯 Who This Is For

**Primary — Active Urban FSD Subscribers**
Drivers using FSD in urban or suburban environments at least three times per week. This is the highest-churn-risk segment and the most valuable source of training miles. The Confidence Layer is built for them first.

**Secondary — All Active FSD Subscribers**
Expansion phase after primary cohort metrics stabilize.

**Long-term — New Subscriber Acquisition**
The Confidence Layer becomes a differentiator against BYD, Waymo, and other ADAS competitors at the top of the funnel once behavioral validation is complete.

---

### ⚠️ The Problem

Tesla's Full Self-Driving (Supervised) is a $99/month subscription that can handle unprotected left turns, lane merges, 4-way stops, and pedestrian crosswalks in cities. The capability is there. The problem is something else.

FSD executes every high-risk maneuver without telling the driver what it's about to do. No preview, no signal, no explanation. The car just acts. For many drivers, one surprising maneuver — a sharp turn they didn't see coming, an unexpected hold at an intersection — is enough to permanently reset their trust baseline. They go back to holding the wheel the whole time. They stop using FSD in cities. Some cancel the subscription.

We engaged **12 participants** to evaluate this problem directly across the SF Bay Area and Los Angeles in May 2026:

- **9 of 12** said they'd trust FSD more if it showed them what it was about to do — with zero change to driving performance
- **3 of 12** were actively considering canceling — unpredictability was the reason every time
- Urban FSD initiation rates lag highway rates by **~25 percentage points** — not a capability gap, a trust gap

The binding constraint on FSD adoption right now is not capability. It is predictability. Drivers intervene because they cannot read the system. This is the gap the Confidence Layer closes.

> Lee & See (2004) showed that trust in automation only develops when users can predict and explain a system's behavior. FSD currently gives drivers neither — especially in the urban maneuvers where trust matters most.

---

### ✅ The Solution

The FSD Confidence Layer makes FSD legible. Two layers ship together inside the existing FSD interface — no new hardware, over-the-air deployment.

Nothing about the underlying FSD driving behavior changes. This is a trust feature, not a driving feature.

> *"Earning trust is not a marketing challenge, it is an engineering and design challenge. FSD Confidence Layer is the first step toward making FSD behavior as readable as a traffic light. When drivers can anticipate what the system is going to do, they engage it more often, in more complex environments, and that engagement makes the system better for every driver on the fleet."*
> — VP of Autopilot Software, Tesla

---

## 🔑 The Two Layers

### Layer 1 — 🟢🟡🔴 Confidence Signal *(Primary)*

A three-tier indicator shown on the center display 1–3 seconds before the maneuver. Sourced directly from FSD's internal scene-uncertainty score — not a separate model, not a post-hoc reconstruction.

| Signal | Level | Alert Copy | Audio |
|:------:|-------|-----------|-------|
| 🟢 | **High** | Path preview only. No alert. | None |
| 🟡 | **Medium** | "Prepare to assist." | Soft chime (1×) |
| 🔴 | **Low** | "Prepare for takeover." | Prominent tone (2×) |

Designed to read at a glance — traffic-light color logic, no manual required. From our research, Participant 12 articulated the intended mental model exactly: *"Red — grab the wheel. Yellow — on alert. Green — calm."* Every one of the 12 participants mapped the colors to the correct action with minimal or no explanation.

Once the maneuver completes, the overlay clears. It does not persist.

---

### Layer 2 — 💬 Natural Language Reasoner / NLR *(Secondary)*

A single plain-English sentence generated entirely on-device by an ~1B parameter distilled model running on HW4's AI4 chip. No cloud connection. P95 latency ≤450ms. Appears alongside the confidence signal as a typewriter-style reveal.

**Real NLR outputs from the prototype:**

```
"Yielding for oncoming traffic, turning left when gap clears ahead."
"Merging left, gap confirmed, slow vehicle ahead in current lane."
"Proceeding through stop, right of way confirmed, you arrived first."
"Slowing, pedestrian stepping into crosswalk ahead on the right."
```

The NLR tells the driver not just *what* FSD is doing, but *why*. It covers only the dominant reason — multi-factor explanations would exceed readable length in the 1–3 second preview window and risk information overload. One-sentence constraint is intentional.

**Validation:** Every OTA push requires passing a 2,500-prompt golden dataset — 0.0% hallucination rate on safety-critical scenarios, hard gate with no exceptions. If the model's own softmax confidence falls below a minimum threshold at runtime, the NLR suppresses its output entirely rather than showing a low-confidence guess.

> The driver remains legally responsible for the vehicle at all times regardless of signal color or NLR output. This is a supervision aid, not an autonomy upgrade.

---

## 🚗 Scenarios in Scope (MVP)

Four non-highway maneuver types where driver interventions happen most often. The overlay does not fire on highway driving, lane-keeping, gentle curves, traffic-light stops on known green phases, or parking.

| # | Scenario | Confidence | What FSD Is Doing | NLR Example |
|---|----------|:----------:|-------------------|-------------|
| 1 | **Unprotected Left Turn** | 🔴 Low | Green light, oncoming traffic, FSD holds for gap then turns | *"Yielding for oncoming traffic, turning left when gap clears ahead."* |
| 2 | **Highway Lane Merge** | 🟢 High | Same-direction lanes, clear gap, FSD merges past slow vehicle | *"Merging left, gap confirmed, slow vehicle ahead in current lane."* |
| 3 | **4-Way Stop** | 🟡 Medium | Right of way confirmed, FSD stops behind line then proceeds | *"Proceeding through stop, right of way confirmed, you arrived first."* |
| 4 | **Pedestrian Crosswalk** | 🟡 Medium | Pedestrian steps in, FSD decelerates and holds, then continues | *"Slowing, pedestrian stepping into crosswalk ahead on the right."* |

| Maneuver | Activation Lead Time |
|----------|---------------------|
| Unprotected left turn | 1.5 – 2.0 seconds before initiation |
| Unprotected intersection entry | 1.2 – 1.8 seconds before initiation |
| Lane change / merge (vehicle in gap) | 1.8 – 2.5 seconds before initiation |
| Complex urban intersection / 4-way stop | 1.5 – 2.0 seconds before initiation |

---

## 🖥 How to Use the Prototype

**Open:** https://preeminent-sherbet-e3685d.netlify.app/

Single-file HTML5/Canvas app. No install, no build step, no server. Works in any browser.

**Step 1 — Pick a scenario**
Select one of the four tabs. Each represents a real FSD maneuver with its rated confidence level.

**Step 2 — Toggle the FSD Layer**
Switch between **OFF** (current FSD — no signal, no preview, the car just acts) and **ON** (Confidence Layer active). This is the before/after comparison.

**Step 3 — Run the Simulation**
Hit **Run Simulation** to trigger the 4-phase sequence: approach → confidence signal fires → NLR types out → maneuver executes → overlay clears.

**Watch the 2-min walkthrough:** https://www.loom.com/share/233a8c3e993e4dc69fec0993fe3004f3

---

## 📊 Market Research & Validation

### Competitive Landscape

No production vehicle today provides real-time, pre-maneuver intent communication with natural language reasoning. That is the gap.

| Competitor | Product | Where They Fall Short |
|------------|---------|----------------------|
| **Waymo** | Fully autonomous (L4) | Not a consumer subscription; geofenced metro areas only; no urban ADAS comparable |
| **BYD God's Eye** | Highway + Urban ADAS | Standard across 21 models, ~150M km/day of training data — but zero confidence signal or NLR for the driver |
| **GM Super Cruise** | Highway L2+ | Strong driver attention monitoring; highway only; no urban confidence coverage |
| **Mercedes Drive Pilot** | L3 certified | Only L3-certified system in limited U.S. states; speed-capped, geofenced, hardware-dependent |

Tesla's defensible advantage here is not the overlay itself — any competitor could ship a similar interface in months. The moat is **calibration quality**. A confidence signal only has value if it is accurate, and accuracy depends on the model and the training data behind it. Tesla's 9+ billion FSD miles provide a calibration advantage that competitors cannot replicate at comparable scale in unconstrained real-world environments. The feature gets more valuable as the model improves, and the model improves fastest with the most engaged subscribers.

### What Users Told Us

12 participants · SF Bay Area + Los Angeles · May 2026
8 of 12 drive daily · 9 of 12 use ADAS features regularly · range from non-FSD users to active daily subscribers

> *"A little overwhelming. But I can see the car speeding up, so I need to be observant. I think I'd rely mostly on the color — that's the quickest thing to process while you're watching the road."*
> — Veronica, non-FSD user, Session 1

**Quantitative Snapshot**

| Metric | Result |
|--------|--------|
| Avg. comfort score vs. no overlay (1–10) | **6.2** · Range: 1–10 · Median: 7 |
| Color coding intuitive immediately | **8 of 12 (67%)** — yes, instantly |
| Would read NLR while driving | 1 actively · 6 situationally · 4 no · 1 maybe |
| WTP impact on FSD subscription | 4 slightly positive · 8 no change · 1 would not use FSD |
| Top unprompted improvement request | Audio / haptic cue for LOW confidence — raised independently by 3 participants |

**What Participants Agreed On**

**1. Color is the primary signal.**
Every participant mapped green/amber/red to the correct action — with minimal or no explanation. No participant failed to understand it. The color layer is the non-negotiable core of the feature. Participant 12: *"Red — grab the wheel. Yellow — on alert. Green — calm."*

**2. Transparency before the maneuver feels meaningful.**
Across all experience levels, knowing what FSD was planning before it acted was valued. Words like "transparent," "human-like," and "practical" came up repeatedly — even from participants who said they wouldn't use FSD themselves.

**3. NLR is useful in principle, distracting in practice.**
Only 1 of 12 said they'd actively read it while driving. 4 said they wouldn't at all. Participants who found it too distracting often followed up by asking for exactly what the NLR provides — a contextual reason for why confidence changed. The content is wanted; always-on text delivery is not.

**4. Audio and haptic feedback is a clear unmet need.**
Three participants independently raised the same design gap — unprompted, across different segments. A visual-only alert is insufficient when eyes should be on the road. This is the clearest product gap this round of research identified.

**5. The feature retains subscribers — it doesn't acquire new ones.**
9 of 12 said it didn't change their willingness to subscribe. Zero said it would convert a non-subscriber. This is consistent with the experiment hypothesis: the Confidence Layer reduces unnecessary disengagement among existing users, not a top-of-funnel acquisition driver.

**Where Participants Diverged**

Participant 8 (experienced FSD user) gave a comfort score of 1 — lowest in the cohort — because they expected FSD to always project 100% confidence. Seeing Medium or Low actively decreased their trust. This is the automation-bias risk segment and the most critical group to monitor in the severe-interventions guardrail.

Counterintuitively, the two highest comfort scores (10 each) came from a light FSD user and a non-FSD user. The feature resonates more with drivers building trust from a low baseline than with experienced users whose expectations are already fixed.

**Design Implications**

1. **Lock in the color signal as-is** — behavioral responses were exactly as intended across all 12 participants
2. **Treat NLR as opt-in or situational**, not always-on text — reduces cognitive load for those who find it distracting
3. **Scope audio/haptic for LOW confidence to Phase 1.5** — three unprompted, independent mentions make this the clearest near-term product gap

### User Stories & JTBD

| User Story | Job To Be Done |
|------------|---------------|
| As an FSD subscriber in the city, I want to see the confidence level before the car makes a move, so I can stop holding the wheel the whole time. | Help me feel in control without requiring me to manually drive. |
| As someone who's been surprised by FSD before, I want a plain-English reason for what the car did, so I can rebuild my mental model instead of permanently lowering my trust. | When the system behaves unexpectedly, help me understand it instead of avoiding it. |
| As a driver paying $99/month, I want the signal early enough to decide whether to intervene, so my attention goes to the moments that actually matter. | Help me direct vigilance to the right moments rather than staying alert the entire drive. |

---

## 🗺 Product Strategy & Prioritization

### MVP Scope

V1 is the two-layer system — Confidence Signal and NLR — covering the four highest-frequency non-highway maneuver types. Every scope decision comes back to one question: does it serve the 1–3 second window before a complex urban maneuver, without changing anything about FSD's underlying driving behavior?

**What is explicitly out of scope for V1:**
- Highway driving — already the most consistent FSD performance, not the binding constraint
- Changes to underlying FSD driving behavior or capability
- Any framing that implies unsupervised operation
- Hardware-dependent features beyond NLR on HW4
- Customer-facing pricing changes — ships free to existing subscribers
- Multi-language NLR — English-only initial rollout; international requires separate regulatory approval
- Always-on or continuous overlay — fires only on classified high-risk maneuvers to preserve signal value

### Key Features to Test

| Feature | What We're Testing |
|---------|--------------------|
| **Confidence Signal (color)** | Does HIGH / MEDIUM / LOW read clearly at a glance, and does the color coding feel intuitive without any explanation? |
| **NLR Explanation** | Does the one-sentence reason feel accurate and trustworthy relative to what the car is actually doing? |
| **Timing & Information Load** | Does receiving both layers together in the 1–3 second window feel helpful, or overwhelming? |

### RICE Prioritization

| Feature | Reach | Impact | Confidence | Effort | Score | Decision |
|---------|:-----:|:------:|:----------:|:------:|:-----:|:--------:|
| Confidence Signal | 9 | 4 | 0.9 | 2 | **16.2** | ✅ V1 |
| NLR (text on display) | 8 | 3 | 0.7 | 4 | **4.2** | ✅ V1 |
| Audio/haptic for LOW | 7 | 3 | 0.8 | 3 | **5.6** | Phase 1.5 |
| NLR (voice output) | 7 | 3 | 0.5 | 5 | **2.1** | ❌ Phase 2 |
| Highway maneuver coverage | 7 | 2 | 0.6 | 3 | **2.8** | ❌ Phase 2 |
| HUD / windshield projection | 6 | 4 | 0.4 | 9 | **1.1** | ❌ Phase 2 |
| Multi-language NLR | 5 | 2 | 0.8 | 3 | **2.7** | ❌ Phase 2 |

*Reach (0–10) · Impact (1–5) · Confidence (0–1) · Effort = engineering weeks*

### A/B Experiment Design

28-day experiment · 3 arms · staged OTA rollout (1% Week 1 → 5% Week 2 → 10% end Week 2 → full eligible fleet if no guardrail violations)

| Arm | What Users Get |
|-----|---------------|
| **Control** | Current FSD — no confidence signal, no NLR, no overlay |
| **T1** | Confidence Signal only, 1–3 seconds before each high-risk maneuver |
| **T2** | Confidence Signal + NLR explanation together |

T1 vs. T2 isolates the marginal value of the NLR explanation on top of the confidence signal alone — that's the central product hypothesis. Randomization is at the vehicle level to avoid within-vehicle treatment contamination.

---

## 📈 Success Metrics & Go-To-Market

### Primary Metric

**Driver disengagements per 1,000 non-highway miles.** Highway miles excluded — FSD disengagement behavior there is already near ceiling. The variance of interest is in urban and suburban mixed-traffic scenarios where the overlay fires. Target: ≥15% reduction in T2 vs. Control at 95% statistical confidence. Experiment window: 28 days, 35-day extension available if significance not reached.

### Supporting Metrics

| Metric | Target |
|--------|--------|
| Trip engagement rate | +10–15% |
| Average FSD engagement duration per trip | Directional increase |
| User-reported distraction events (post-drive in-app) | No meaningful increase vs. baseline |

### Checkpoints

| Day | Review Type |
|-----|------------|
| Day 7 | Interim safety check |
| Day 14 | Behavioral check |
| Day 28 | Full readout — primary metric + all guardrails |

**Long-horizon indicators:** Subscription retention and share of miles driven on FSD among active subscribers. These are the durable signals that trust has actually been established, not just temporarily improved.

### Guardrails

| Guardrail | Threshold | Consequence |
|-----------|-----------|-------------|
| Subscription cancellation rate | Must not increase >2% vs. baseline | Rollback |
| False positive confidence rate | <2% of high-confidence maneuvers | Hard rollback — revenue protection as much as safety |
| Distraction events (driver monitoring camera) | No meaningful increase vs. baseline | Immediate pause |
| NLR hallucination rate | ≤0.5% overall · **0.0% on safety-critical scenarios** | No OTA push proceeds until both gates clear |

### Strategic Tie to Robotaxi

The Cybercab depends on regulatory approval for unsupervised operation, which depends on public trust and a strong safety record. Higher FSD engagement among current subscribers generates more diverse training miles, which strengthens the safety case to NHTSA.

This is not a robotaxi feature. It accelerates the data flywheel that makes the robotaxi timeline credible. Every percentage point of additional engagement from existing subscribers compounds.

> BYD God's Eye is generating ~150M km of training data per day across 21 models. Tesla's 9+ billion FSD miles give the Confidence Layer's calibration model an accuracy advantage competitors can't replicate at this scale in unconstrained real-world environments. The flywheel is the moat.

### GTM

**Phase 1 — Staged OTA (Weeks 1–8)**
1% of eligible fleet → 5% → 10% → full eligible fleet if no guardrail violations. Opt-in via Tesla app for beta. No press release until 10% threshold is cleared and guardrails are clean.

**Phase 2 — Full Rollout**
Standard OTA to all Model Y on FSD v13.2+ with HW4. No hardware change, no app store approval. Ships free to existing subscribers. External comms emphasize driver transparency and supervision — not autonomy.

### Product Roadmap

Three-phase driver-AI communication architecture:

**Phase 1 — Current: Unidirectional Intent Communication**
FSD shows the driver what it is planning and why, 1–3 seconds before every high-risk maneuver. Confidence Signal + NLR, OTA to all HW4 vehicles.

**Phase 1.5 — Near Term: Non-Visual Modality**
Audio cue or haptic pulse for LOW confidence scenarios. Three independent unprompted mentions in user research make this the clearest product gap from this round.

**Phase 2 — 12–18 Months: Bidirectional Dialogue**
Drivers can query FSD via voice ("Why are we slowing down?") and receive a contextualized answer drawn from the vehicle's real-time scene understanding.

**Phase 3 — 24+ Months: Personalized Confidence Thresholds**
Overlay frequency and thresholds calibrated to individual driver trust profiles built from longitudinal behavioral history. High-trust drivers see fewer prompts; new drivers see more detail.

---

## 👥 Team

| Name | School | Contributions |
|------|--------|--------------|
| **Prarthana Patel** | UCLA Anderson | Interactive prototype, GitHub repository, Eval summary |
| **Rafael Manansala** | UCLA Anderson | PR-FAQ, product narrative, Interactive prototype |
| **Abhisek Jose Selvakumar** | UCLA Anderson | Source of truth, research synthesis, Loom walkthrough |

---

## 📖 Sources

### Academic Research
- Lee, J. D., & See, K. A. (2004). Trust in automation: Designing for appropriate reliance. *Human Factors, 46*(1), 50–80.
- Hoff, K. A., & Bashir, M. (2015). Trust in automation: Integrating empirical evidence on factors that influence trust. *Human Factors, 57*(3), 407–434.
- Endsley, M. R. (1995). Toward a theory of situation awareness in dynamic systems. *Human Factors, 37*(1), 32–64.

### Industry & Regulatory
- NHTSA EA22-002 (2022). *Tesla Autopilot Forward Collision Avoidance Assist Investigation.* National Highway Traffic Safety Administration.
- SAE International (2021). *Taxonomy and Definitions for Terms Related to Driving Automation Systems for On-Road Motor Vehicles* (SAE J3016).
- Tesla, Inc. (2024). *Full Self-Driving (Supervised) Release Notes v13.2.*
- BYD Co. Ltd. (2025). *God's Eye ADAS Platform Launch.* Shenzhen.

### Primary Research
- FSD Confidence Layer User Interviews (May 2026). 12 participants evaluated the prototype — 1 full transcript + 11 structured summaries capturing comfort lift score, NLR response, WTP impact, and improvement suggestions. SF Bay Area and Los Angeles. Conducted by Prarthana Patel, Rafael Manansala, and Abhisek Jose Selvakumar. UCLA Anderson, MGMT 275.
- FSD Confidence Layer Prototype Study (May 2026). https://forms.gle/CjJcSuY6gaznYfnT6
- FSD Confidence Layer Source of Truth (May 2026). Canonical product reference document. Authored by Abhisek Jose Selvakumar. UCLA Anderson, MGMT 275.
- FSD Confidence Layer PR-FAQ (May 2026). Press release and internal/external FAQ. Authored by Rafael Manansala. UCLA Anderson, MGMT 275.

---

<div align="center">

*MGMT 275 · UCLA Anderson School of Management · May 2026*

*Prarthana Patel · Rafael Manansala · Abhisek Jose Selvakumar*

</div>
