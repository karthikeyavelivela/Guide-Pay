# Guide-Pay

### Parametric Income Insurance for India's Gig Delivery Workers

> Auto-pays when floods, outages, or curfews stop delivery workers from earning.
> **Zero claims. Zero paperwork.**

---

![Status](https://img.shields.io/badge/status-prototype-blue)
![AI Powered](https://img.shields.io/badge/AI-XGBoost%20%7C%20Isolation%20Forest-orange)
![Backend](https://img.shields.io/badge/backend-FastAPI-green)
![Frontend](https://img.shields.io/badge/frontend-React%2018-blue)
![Database](https://img.shields.io/badge/database-MongoDB-darkgreen)
![Payments](https://img.shields.io/badge/payments-Razorpay-purple)

**Team SentinelX · KL University**
Guidewire **DEVTrails 2026**

---

# Problem

Ravi Kumar, 27, delivers for Zepto in **Kondapur, Hyderabad**.

* Earns **₹800/day**
* Works across **Zepto + Swiggy**
* No insurance coverage

**July 2024**

IMD issues a **Red Alert**.
Flash floods hit Hyderabad.

Delivery platforms **suspend operations**.

Ravi loses **₹640 in a single day**.

No claim.
No coverage.
No income protection.

---

## Market Reality

* **12+ million** gig delivery workers in India
* Workers lose **20–30% income during monsoon seasons**
* **Zero parametric insurance products exist**
* Estimated **₹4,200 crore annual uninsured exposure**

---

# Solution

**Guide-Pay** is a **parametric income insurance platform** designed specifically for gig delivery workers.

The platform:

1. **Predicts disruptions** before they occur using AI
2. **Monitors real-world triggers every 15 minutes**
3. **Verifies worker activity automatically**
4. **Detects fraud using multi-worker event correlation**
5. **Pays workers automatically via UPI**

Workers **never file claims**.

Payout happens automatically when disruption conditions are met.

---

# Parametric Triggers

Guide-Pay activates payouts when **objective external events occur**.

| Trigger            | Data Source             | Threshold           | Payout |
| ------------------ | ----------------------- | ------------------- | ------ |
| Flood / Rain Alert | IMD SACHET RSS          | Orange / Red alert  | 100%   |
| Platform Outage    | Downdetector            | >500 outage reports | 75%    |
| Government Curfew  | District administration | Official order      | 100%   |

All triggers are:

* third-party verified
* binary
* geofenced within **5km worker zone**
* polled every **15 minutes**

---

# Why AQI Is Excluded

Air pollution does not **halt platform operations**.

Workers may choose to stop working voluntarily.

Parametric insurance requires a strict chain:

**objective trigger → verified income loss → payout**

AQI fails to guarantee **income interruption**, so it is excluded.

---

# Target Persona

**Ravi Kumar, 27**
Delivery partner in Kondapur, Hyderabad

Works on:

* Zepto
* Swiggy

Income:

₹700 – ₹900 per day

Loss during floods:

₹500 – ₹640 per day

Primary target cities:

* Hyderabad
* Mumbai
* Chennai
* Bengaluru

---

# Premium Model

Weekly premium is calculated using zone risk and worker reliability.
```text
Weekly Premium
= ₹49 × Zone Risk Multiplier × Worker Risk Adjustment
```

### Zone Risk Multiplier

Range: **0.8 → 1.4**

Risk weights:

* Flood risk → 50%
* Curfew risk → 30%
* Platform outage risk → 20%

### Worker Risk Adjustment

Range: **0.85 → 1.15**

| Risk Score | Multiplier |
| ---------- | ---------- |
| >0.75      | 0.85       |
| 0.50–0.75  | 1.00       |
| <0.50      | 1.15       |

---

# Example Premiums

| City                    | Base | Zone Adj | Worker Adj | Final |
| ----------------------- | ---- | -------- | ---------- | ----- |
| Kondapur (Hyderabad)    | ₹49  | ×1.40    | ×0.85      | ₹58   |
| Kurla (Mumbai)          | ₹49  | ×1.40    | ×1.00      | ₹69   |
| Koramangala (Bengaluru) | ₹49  | ×1.10    | ×1.00      | ₹54   |

---

# AI / ML Components

Guide-Pay integrates **5 AI modules**.

---

## Predictive Disruption Engine

Predicts floods before they happen.

Model: **XGBoost Regressor**

Inputs:

* IMD rainfall forecast
* Zone elevation (SRTM DEM)
* NDMA flood history
* Monsoon intensity index
* NASA SMAP soil moisture

Output:
```text
next_24h_disruption_probability
```

Example:
```text
Kondapur Flood Risk → 78%
32 workers affected
₹19,200 potential exposure
```

---

## Platform Activity Verification

Ensures the worker was **actively working before payout**.

Data source: Mock delivery platform API.

Logic:

| Last Order Age | Payout  |
| -------------- | ------- |
| <2 hrs         | 100%    |
| 2–4 hrs        | 75%     |
| >6 hrs         | Flagged |

---

## Worker Risk Score

Insurance **credit score for gig workers**.

Range: `0.0 → 1.0`

Score components:

* Delivery activity → 35%
* Claim frequency → 30%
* Fraud signals → 20%
* Active hours → 15%

Example:
```text
Score: 0.82
Risk Tier: Low
Premium multiplier: 0.85
Weekly savings: ₹7
```

---

## Multi-Worker Event Correlation

Confirms disruption events collectively.

| Zone Claims  | Result    |
| ------------ | --------- |
| >60% workers | Confirmed |
| <10% workers | Flagged   |

Example:
```text
Kondapur Flood
28 / 33 workers → 84%
Auto-approved
```

---

## Fraud Detection Engine

Two stages:

**Phase 2:** Rule-based detection

**Phase 3:** Isolation Forest model

Fraud checks include:

* GPS mismatch
* duplicate claims
* claim frequency anomaly
* IP location mismatch
* device inconsistency
* activity verification
* zone correlation ratio

Rule:
```text
Fraud score >0.70 → manual review
Fraud score <0.70 → auto payout
```

---

# Adversarial Defense & Anti-Spoofing Strategy

## Threat Model

A coordinated syndicate of 500 delivery workers using GPS-spoofing apps to fake zone presence during red-alert weather events, triggering mass parametric payouts while physically at home.

---

## 1. Differentiating a Genuine Worker from a GPS Spoofer

Simple GPS coordinates are insufficient. Guide-Pay uses a **6-signal behavioral fingerprint**:

| Signal | Genuine Worker | Spoofer |
|---|---|---|
| `last_order_timestamp` | Recent delivery activity in zone | No orders — platform shows idle |
| Device accelerometer | Movement consistent with vehicle | Stationary despite claimed flood zone |
| Battery drain rate | High (navigation + delivery app running) | Low (phone sitting idle) |
| Network cell tower ID | Matches claimed GPS zone | Mismatch — tower in different area |
| Platform session activity | Active app session, accepting orders | No platform session open |
| Historical zone presence | Worker's delivery history includes zone | Worker never delivered here before |

A spoofer can fake GPS coordinates. They cannot simultaneously fake all six signals. Our fraud score requires at least 4/6 signals to be consistent before approving a payout.

---

## 2. Data Points for Detecting a Coordinated Fraud Ring

Individual claim analysis catches solo fraudsters. Ring detection requires cross-worker correlation:

**Temporal clustering:** If 50+ workers claim the same zone within a 15-minute window, that is a trigger event — not suspicious. If 50+ workers all submit claims within 90 seconds of each other, that is coordinated. No real disruption produces simultaneous human response at that precision.

**Device fingerprint overlap:** Syndicate members often use the same spoofing app and device model. We flag claims where more than 10 workers share identical `user_agent` + `device_model` + `app_version` combinations.

**Payout velocity anomaly:** Isolation Forest model trained on normal claim rate per zone per hour. A 10x spike in claim submissions from a zone with no corresponding IMD alert escalation is an automatic ring flag — payouts frozen pending verification.

**Zero delivery history in claimed zone:** If a worker has never completed a delivery in the zone they are claiming from, that is a primary fraud signal.

**The 15-worker threshold (existing defense):** Our Multi-Worker Event Correlation module already requires 15+ workers in the same zone to confirm an event. A ring of 500 simultaneous claims from a zone with no matching platform outage or IMD data is the loudest fraud signal possible — it triggers an immediate freeze, not an approval.

---

## 3. UX Balance: Flagged Claims Without Punishing Honest Workers

**Green — Auto-approve (4/6 signals consistent):**
Worker receives payout within 2 hours. No friction.

**Amber — Soft verification (2–3/6 signals inconsistent):**
Worker receives a single in-app prompt: *"We noticed unusual network activity in your area. Tap to confirm your location."* One-tap confirmation with live photo capture (timestamp + GPS). If confirmed, payout processes within 4 hours. No claim denied at this stage.

**Red — Manual review (ring flag triggered):**
Payout held. Worker notified: *"Your claim is under review due to high activity in your zone. Expected resolution: 24 hours."* Worker is never told they are suspected of fraud — only that volume is high.

**Honest worker protection rule:** First-time flagged workers with a Worker Risk Score below 0.3 automatically receive Amber treatment, never Red — even if signals are inconsistent. A real flood causes genuine network drops. We do not penalise workers for bad connectivity during the exact event we are insuring against.

---

## Why Our Architecture Was Already Resistant

This attack scenario was anticipated in the original design:

* **Activity Verification Module** already checks `last_order_timestamp` — a worker at home has no recent orders
* **Multi-Worker Event Correlation** already uses Isolation Forest on zone × time — ring behavior is an anomaly by definition
* **Worker Risk Score** already tracks claim frequency — first fraudulent claim raises score, second gets flagged automatically

The GPS spoofing attack does not break Guide-Pay. It activates defenses we already built.

---

# System Architecture
```text
IMD SACHET RSS
       │
       ▼
Trigger Engine (15 min polling)
       │
       ▼
Event Detection
       │
       ▼
Worker Activity Verification
       │
       ▼
Fraud Detection Engine
       │
       ▼
Zone Correlation Check
       │
       ▼
Auto Claim Approval
       │
       ▼
UPI Payout via Razorpay
```

---

# Tech Stack

| Layer            | Technology             |
| ---------------- | ---------------------- |
| Frontend         | React 18 + TailwindCSS |
| Backend          | FastAPI                |
| Database         | MongoDB                |
| Authentication   | Firebase OTP           |
| ML Models        | XGBoost                |
| Fraud Detection  | Isolation Forest       |
| Trigger Data     | IMD SACHET             |
| Outage Data      | Downdetector           |
| Maps             | Google Maps API        |
| Payments         | Razorpay               |
| Frontend Hosting | Vercel                 |
| Backend Hosting  | Railway                |

---

# Automated Claim Flow
```text
Every 15 minutes

1. Trigger engine polls IMD + Downdetector
2. Detect disruption event
3. Create TriggerEvent in MongoDB
4. Find workers in geofenced zone
5. Verify worker activity
6. Run fraud detection
7. Calculate correlation ratio
8. Approve payout
9. Send UPI transfer via Razorpay
```

Workers receive payouts **within 2 hours**.

---

# Market Size

| Metric            | Value         |
| ----------------- | ------------- |
| Gig workers India | ~12 million   |
| Tier-1 workers    | ~4.2 million  |
| TAM               | ₹30,576 crore |
| SAM               | ₹6,370 crore  |
| SOM (Year 1)      | ₹9.55 crore   |
| Target Loss Ratio | 65%           |

---

# Roadmap

### Phase 1 — Seed

Architecture design
AI module specification
Prototype

### Phase 2 — Scale

Authentication
Trigger detection engine
Activity verification
Fraud rules

### Phase 3 — Soar

Predictive ML engine
Isolation Forest fraud detection
Razorpay payouts
Full dashboards

---

# Demo

**Demo Video:** [YouTube — unlisted link](PASTE_YOUTUBE_LINK_HERE)

**Prototype:** [Live Demo](PASTE_VERCEL_LINK_HERE)

**Repository:** [GitHub](PASTE_GITHUB_REPO_LINK_HERE)

---

# Team SentinelX

KL University
Guidewire DEVTrails 2026

---

# Vision

Guide-Pay transforms gig insurance from **reactive claims processing** to **predictive income protection**.

Instead of waiting for workers to report losses,

the system **predicts disruptions, verifies activity, and pays automatically**.

---

**Guide-Pay**

*We don't just react to disruptions. We predict them.*