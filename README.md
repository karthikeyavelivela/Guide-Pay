# Guide-Pay

### Parametric Income Insurance for India's Gig Delivery Workers

> Auto-pays when floods, outages, or curfews stop delivery workers from earning.
> **Zero claims. Zero paperwork. Payout in under 2 hours.**

---

![Status](https://img.shields.io/badge/status-Phase%201%20Complete-brightgreen)
![AI Powered](https://img.shields.io/badge/AI-XGBoost%20%7C%20Isolation%20Forest-orange)
![Backend](https://img.shields.io/badge/backend-FastAPI-green)
![Frontend](https://img.shields.io/badge/frontend-React%2018-blue)
![Database](https://img.shields.io/badge/database-MongoDB-darkgreen)
![Payments](https://img.shields.io/badge/payments-Razorpay-purple)
![Phase](https://img.shields.io/badge/Guidewire-DEVTrails%202026-blue)

**Team SentinelX · KL University**
Guidewire **DEVTrails 2026** — Phase 1 Submission

---

## Demo

| | Link |
|---|---|
| 📹 Demo Video | [Watch on YouTube](PASTE_YOUTUBE_LINK_HERE) |
| 🖥 Live Prototype | [Open Demo](PASTE_VERCEL_LINK_HERE) |
| 💻 Repository | [GitHub](PASTE_GITHUB_REPO_LINK_HERE) |

---

## The Problem

**Ravi Kumar, 27.** Delivers for Zepto in Kondapur, Hyderabad.
Earns ₹800/day. Works across Zepto + Swiggy. Zero insurance.

**July 2024.** IMD issues a Red Alert. Flash floods hit Hyderabad.
Zepto suspends all operations.

Ravi loses **₹640 in a single day.**

He doesn't file a claim.
**There is no claim to file.**

---

### Market Reality

| Metric | Value |
|---|---|
| Gig delivery workers in India | 12+ million |
| Income lost per worker per monsoon | 20–30% of monthly earnings |
| Parametric insurance products for gig workers | **Zero** |
| Annual uninsured income loss | **₹4,200 crore** |

This is not a feature gap. It is a structural void.

---

## Solution

**Guide-Pay** is a parametric income insurance platform built exclusively for gig delivery workers.
```
Worker pays ₹49/week
         ↓
External disruption is detected automatically
         ↓
Worker activity is verified (was Ravi actually working?)
         ↓
Fraud engine confirms legitimacy
         ↓
₹600 credited to UPI within 2 hours
         ↓
Ravi filed nothing. Ravi pressed nothing.
```

---

## Parametric Triggers

Three triggers. All binary. All third-party verified. All geofenced to the worker's 5km zone.

| Trigger | Data Source | Threshold | Payout Cap |
|---|---|---|---|
| IMD Flood / Rain Alert | IMD SACHET RSS | Orange or Red alert | 100% |
| Platform App Outage | Downdetector + platform status API | >500 outage reports | 75% |
| Government Curfew / Section 144 | District admin API | Official order issued | 100% |

Polled every **15 minutes**, 24/7.

---

## Why AQI Is Deliberately Excluded

Every competing team included AQI as a trigger. We excluded it — and here is why that makes Guide-Pay stronger.

Parametric insurance requires a strict causation chain:
```
Objective External Event → Platform Halts → Worker Cannot Earn → Payout
```

AQI fails this test. Platforms like Zepto and Swiggy do **not** halt operations during pollution events. High-AQI days show normal or elevated order volumes due to homebound demand. Workers who stop during poor air quality are making a voluntary choice — not experiencing a verified income interruption.

Insuring voluntary behavior creates:
- **Moral hazard** — workers reduce shifts on bad-air days to trigger payouts
- **Adverse selection** — workers with respiratory conditions over-enroll
- **Unverifiable causation** — no external API confirms income was actually lost

AQI exclusion is an actuarial decision. It makes our loss ratio defensible.

---

## Premium Model
```
Weekly Premium = ₹49 × Zone Risk Multiplier × Worker Risk Adjustment
```

### Zone Risk Multiplier (0.8 → 1.4)

Derived from IMD historical flood frequency × SRTM zone elevation data.

| Component | Weight |
|---|---|
| Flood risk | 50% |
| Curfew risk | 30% |
| Platform outage risk | 20% |

### Worker Risk Adjustment (0.85 → 1.15)

Behavior-based pricing. Reliable workers pay less.

| Risk Score | Multiplier | Effect |
|---|---|---|
| > 0.75 | 0.85× | Discount — Low Risk |
| 0.50 – 0.75 | 1.00× | Base rate — Medium Risk |
| < 0.50 | 1.15× | Surcharge — High Risk |

### Example Premiums

| Worker Zone | Base | Zone Adj | Worker Adj | Final |
|---|---|---|---|---|
| Kondapur, Hyderabad | ₹49 | ×1.40 | ×0.85 | **₹58** |
| Kurla, Mumbai | ₹49 | ×1.40 | ×1.00 | **₹69** |
| Koramangala, Bengaluru | ₹49 | ×1.10 | ×1.00 | **₹54** |

**Coverage cap: ₹600/week. Target loss ratio: 65%.**

---

## 5 AI Modules

---

### Module 1 — Predictive Disruption Engine

Predicts flood disruption 24 hours before it occurs.

**Model:** XGBoost Regressor

**Inputs:**
- IMD 24h rainfall forecast
- Zone elevation (SRTM DEM)
- NDMA historical flood events (2015–2024)
- Monsoon intensity index
- NASA SMAP soil moisture

**Output:**
```
next_24h_disruption_probability per zone (0.0 → 1.0)
```

**Example:**
```
Kondapur Flood Risk → 78%
32 workers affected
₹19,200 potential exposure
Coverage auto-extended → worker notified
```

When probability exceeds 0.70, the worker receives a preemptive warning and coverage is automatically extended. The worker does nothing.

---

### Module 2 — Platform Activity Verification

Answers the critical question: **Was the worker actually working when the disruption hit?**

**Data source:** Mock platform API (`last_order_timestamp`)

| Last Order Age at Trigger Time | Payout |
|---|---|
| < 2 hours | 100% |
| 2 – 6 hours | 75% |
| > 6 hours | Flagged — no payout |

A worker who was already offline before the flood is not eligible. This is the primary defense against cold-start fraudulent claims.

---

### Module 3 — Real-Time Disruption Heat Map

Interactive India map on the admin dashboard. Color-coded zone markers:

- 🔴 Red — active flood alert
- 🟠 Amber — platform outage
- 🟣 Purple — government curfew
- 🟢 Green — clear

Each marker shows: workers affected, payout exposure (₹), event timestamp.

This transforms Guide-Pay from a worker-facing insurance product into a **platform intelligence system** — enabling logistics pre-positioning and enterprise data licensing as a second revenue surface.

---

### Module 4 — Worker Risk Score

Insurance credit score for gig workers. Range: 0.0 → 1.0.

| Signal | Weight |
|---|---|
| Delivery activity (orders/day vs. platform average) | 35% |
| Claim frequency (claims per 100 active days) | 30% |
| Fraud risk signals | 20% |
| Active hours ratio | 15% |

Low-risk workers (score > 0.75) receive a **0.85× premium multiplier** — a real financial incentive for consistent, honest behavior. This is behavior-based pricing. Standard InsurTech practice. No competing team has built it.

---

### Module 5 — Multi-Worker Event Correlation

The most important fraud signal. Genuine disruptions are geographically correlated. Fraud is not.

**Model:** Isolation Forest (scikit-learn) on zone × time feature space

| Zone Claim Ratio | Result |
|---|---|
| > 60% of workers in zone | Event confirmed — auto-approve |
| < 10% of workers in zone | Anomaly — flag all claimers |

**Example:**
```
Kondapur Flood — 28 of 33 workers claimed (84%)
→ Event confirmed
→ Fraud risk reduced across all 28 claims
→ Auto-approved
```

A single worker claiming a flood in a zone where 32 other workers show no disruption is a fraud signal, not a payout.

---

## Adversarial Defense & Anti-Spoofing Strategy

### Threat Model

A coordinated syndicate of 500 delivery workers using GPS-spoofing applications to fake zone presence during red-alert weather events — triggering mass false payouts while physically at home.

---

### 1. Differentiating a Genuine Worker from a GPS Spoofer

Simple GPS coordinates are dead. Guide-Pay uses a **6-signal behavioral fingerprint**:

| Signal | Genuine Worker | GPS Spoofer |
|---|---|---|
| `last_order_timestamp` | Recent delivery in zone | Platform shows idle — no orders |
| Device accelerometer | Movement consistent with vehicle | Stationary despite claimed zone |
| Battery drain rate | High — navigation + delivery apps active | Low — phone sitting idle |
| Network cell tower ID | Tower matches claimed GPS zone | Tower mismatch — different area |
| Platform session activity | Active session, accepting/declining orders | No platform session open |
| Historical zone presence | Delivery history includes this zone | Worker has never delivered here |

A spoofer can fake GPS. They cannot simultaneously fake all six signals. Our fraud engine requires **4 of 6 signals consistent** before approving a payout.

---

### 2. Detecting a Coordinated Fraud Ring

Individual analysis catches solo fraudsters. Ring detection requires cross-worker correlation.

**Temporal clustering:** 50+ workers claiming the same zone in a 15-minute window is a real trigger event — expected. 50+ workers submitting claims within 90 seconds of each other is coordinated — no real disruption produces simultaneous human response at that precision.

**Device fingerprint overlap:** Syndicate members share the same spoofing application, device model, and app version. We flag claims where more than 10 workers share identical `user_agent` + `device_model` + `app_version` combinations.

**Payout velocity anomaly:** Isolation Forest trained on normal claim rate per zone per hour. A 10× spike with no corresponding IMD alert escalation triggers an automatic zone freeze — payouts suspended pending manual review.

**Zero delivery history in claimed zone:** A worker who has never completed a delivery in the zone they are claiming from is a primary fraud flag.

**Existing 15-worker threshold:** Our Multi-Worker Event Correlation module already requires 15+ workers in zone to confirm any event. A ring of 500 simultaneous claims from a zone with no IMD or platform data is the loudest fraud signal possible — it triggers an immediate freeze, not an approval.

---

### 3. Flagged Claims Without Punishing Honest Workers

| Status | Trigger Condition | Action |
|---|---|---|
| 🟢 Green — Auto-approve | 4+ of 6 signals consistent | Payout within 2 hours |
| 🟡 Amber — Soft verify | 2–3 of 6 signals inconsistent | Single in-app prompt: one-tap location confirm + timestamped photo. Payout within 4 hours if confirmed. |
| 🔴 Red — Manual review | Ring flag triggered | Payout held 24 hours. Worker notified of "high zone activity" — never told they are suspected. |

**Honest worker protection rule:** Workers with a Risk Score below 0.3 and no prior fraud flags always receive Amber treatment, never Red — even if signals are inconsistent. Real floods cause genuine network drops. We do not penalize workers for bad connectivity during the exact event we are insuring against.

---

### Why Our Architecture Was Already Resistant

This attack scenario was anticipated in the original design. The GPS spoofing attack does not break Guide-Pay — it activates defenses we already built:

- **Activity Verification** checks `last_order_timestamp` — a worker at home has no recent orders
- **Event Correlation** uses Isolation Forest on zone × time — ring behavior is an anomaly by definition  
- **Worker Risk Score** tracks claim frequency — first fraudulent claim raises score, second is auto-flagged

---

## System Architecture
```
IMD SACHET RSS    Downdetector API    Admin Webhook
       │                 │                 │
       └─────────────────┴─────────────────┘
                         │
                         ▼
              Trigger Engine (15 min cron)
                         │
                         ▼
               Event Detection + Geofence
                         │
                         ▼
           Worker Activity Verification
           (last_order_timestamp gate)
                         │
                         ▼
              Fraud Detection Engine
              (7-check rule + Isolation Forest)
                         │
                         ▼
           Multi-Worker Correlation Check
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Auto-Approve           Manual Review
              │
              ▼
     Razorpay UPI Payout
     (under 2 hours)
```

---

## Tech Stack

| Layer | Technology | Reason |
|---|---|---|
| Frontend | React 18 + TailwindCSS | Fast iteration, PWA-ready |
| Backend | FastAPI (Python) | Async — handles concurrent payouts |
| Database | MongoDB | Flexible schema for variable claim types |
| Auth | Firebase OTP | Phone-first, zero documents |
| ML — Premium | XGBoost | Tabular data, fast inference, interpretable |
| ML — Fraud | Isolation Forest (scikit-learn) | Unsupervised anomaly detection — no labeled fraud data needed at launch |
| Flood Trigger | IMD SACHET RSS | Official govt source, free, real-time |
| Outage Trigger | Downdetector + platform status API | Dual-source verification |
| Payments | Razorpay (test mode) | UPI auto-debit + auto-credit, ₹0.01/txn at scale |
| Maps | Google Maps API | Zone geofencing, 5km radius |
| Frontend Deploy | Vercel | Zero-config CI/CD |
| Backend Deploy | Railway | 4× cheaper than AWS for Python ML at this stage |

---

## Automated Claim Flow
```
Every 15 minutes:

Step 1  →  Poll IMD SACHET RSS + Downdetector
Step 2  →  Detect disruption event in any active zone
Step 3  →  Create TriggerEvent document in MongoDB
Step 4  →  Query workers with active policies in geofenced zone
Step 5  →  Verify last_order_timestamp for each worker
Step 6  →  Run 7-check fraud scoring per claim
Step 7  →  Calculate zone correlation ratio
Step 8  →  Auto-approve (score < 0.70) or queue for review (score ≥ 0.70)
Step 9  →  Dispatch UPI transfer via Razorpay SDK
Step 10 →  Push Firebase notification to worker
```

**Target: payout credited within 2 hours of trigger detection.**

---

## Market Size

| Metric | Value | Basis |
|---|---|---|
| Total gig delivery workers | ~12 million | NASSCOM 2024 |
| Tier-1 flood-prone city workers | ~4.2 million | IMD zone overlay |
| TAM | ₹30,576 crore/year | 12M × ₹49/week × 52 |
| SAM | ₹6,370 crore/year | 4.2M workers |
| SOM Year 1 | ₹9.55 crore/year | 210K workers, Hyderabad + Mumbai |
| Target loss ratio | 65% | Parametric micro-insurance benchmark |

---

## Roadmap

### Phase 1 — Seed ✅
- Worker persona research (Ravi Kumar, Hyderabad)
- Parametric trigger design + AQI exclusion rationale
- Premium model with actuarial assumptions
- 5 AI module specifications
- Full interactive prototype (10 screens)
- Adversarial defense + anti-spoofing architecture
- Technical dev blog published on Medium

### Phase 2 — Scale
- Firebase OTP authentication
- Live trigger detection engine (IMD + Downdetector)
- Platform activity verification (mock API)
- Rule-based fraud engine (7 checks)
- Worker + admin dashboard

### Phase 3 — Soar
- XGBoost predictive engine (trained on IMD 2015–2024)
- Isolation Forest fraud model
- Razorpay live UPI payouts
- Real-time heat map
- IRDAI sandbox application

---

## Team SentinelX

KL University · Guidewire DEVTrails 2026

---

## Vision

Guide-Pay transforms gig insurance from reactive claims processing to predictive income protection.

The industry benchmark for claims settlement is 7–10 days.
Our target is **2 hours** — achieved automatically, with no action from the worker.

---

*We don't just react to disruptions. We predict them.*