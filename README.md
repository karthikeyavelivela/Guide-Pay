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
= ₹35 × Zone Risk Multiplier × Worker Risk Adjustment
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
| Kondapur (Hyderabad)    | ₹35  | ×1.40    | ×0.90      | ₹44   |
| Kurla (Mumbai)          | ₹35  | ×1.40    | ×1.00      | ₹49   |
| Koramangala (Bengaluru) | ₹35  | ×1.10    | ×1.00      | ₹39   |

---

# AI / ML Components

Guide-Pay integrates **5 AI modules**.

---

## Predictive Disruption Engine

Predicts floods before they happen.

Model

**XGBoost Regressor**

Inputs

* IMD rainfall forecast
* Zone elevation (SRTM DEM)
* NDMA flood history
* Monsoon intensity index
* NASA SMAP soil moisture

Output

```text
next_24h_disruption_probability
```

Example

```text
Kondapur Flood Risk → 78%
32 workers affected
₹19,200 potential exposure
```

---

## Platform Activity Verification

Ensures the worker was **actively working before payout**.

Data source

Mock delivery platform API.

Logic

| Last Order Age | Payout  |
| -------------- | ------- |
| <2 hrs         | 100%    |
| 2–4 hrs        | 75%     |
| >6 hrs         | Flagged |

---

## Worker Risk Score

Insurance **credit score for gig workers**.

Range

```text
0.0 → 1.0
```

Score components

* Delivery activity → 35%
* Claim frequency → 30%
* Fraud signals → 20%
* Active hours → 15%

Example

```text
Score: 0.82
Risk Tier: Low
Premium multiplier: 0.85
Weekly savings: ₹5
```

---

## Multi-Worker Event Correlation

Confirms disruption events collectively.

| Zone Claims  | Result    |
| ------------ | --------- |
| >60% workers | Confirmed |
| <10% workers | Flagged   |

Example

```text
Kondapur Flood
28 / 33 workers → 84%
Auto-approved
```

---

## Fraud Detection Engine

Two stages:

Phase 2
Rule-based detection

Phase 3
Isolation Forest model

Fraud checks include:

* GPS mismatch
* duplicate claims
* claim frequency anomaly
* IP location mismatch
* device inconsistency
* activity verification
* zone correlation ratio

Rule

```text
Fraud score >0.70 → manual review
Fraud score <0.70 → auto payout
```

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

Demo Video
YouTube link

Prototype
Vercel link

Repository
GitHub repo

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
