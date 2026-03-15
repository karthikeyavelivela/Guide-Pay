# Guide-Pay
### Parametric income insurance for India's gig delivery workers.

> Auto-pays when floods, outages, or curfews stop them from working. Zero claims. Zero paperwork.

**Team SentinelX · KL University · Guidewire DEVTrails 2026**

---

## The Problem

Ravi Kumar, 27, delivers for Zepto in Kondapur, Hyderabad. Earns ₹800/day.

July 2024 — IMD issues a Red Alert. Flash floods hit his zone. Zepto suspends operations. Ravi loses ₹640 in one day. He opens no insurance app. Nothing covers this.

- **12+ million** gig delivery workers in India
- External disruptions cause **20–30% monthly income loss** during monsoon seasons
- **Zero** parametric income insurance products exist for this segment
- Estimated uninsured exposure: **₹4,200 crore / year**

---

## The Solution

Guide-Pay is a parametric income insurance platform that:

1. **Predicts** disruptions before they happen using an AI Forecast Engine
2. **Monitors** 3 binary, third-party verified triggers every 15 minutes
3. **Verifies** the worker was actually working before paying out
4. **Correlates** claims across workers to confirm events and detect fraud
5. **Auto-pays** to worker UPI within 2 hours — worker does nothing

---

## Why We Excluded AQI

Platforms don't halt operations during high AQI. Workers can choose to stay home — but this breaks the parametric causation chain. Parametric insurance requires: **objective trigger → verified income loss → payout**. AQI fails at step 2.

Guide-Pay only covers events that cause a platform-level operational halt: binary, verifiable, zero worker discretion.

---

## The 3 Triggers

| Trigger | Source | Threshold | Payout |
|---------|--------|-----------|--------|
| IMD Flood / Rain Alert | IMD SACHET RSS (official govt) | Orange or Red alert in district | 100% cap |
| Platform App Outage | Downdetector + platform status | >500 reports in 2hr window | 75% cap |
| Government Curfew | Admin verification + district order | Official order in worker's zone | 100% cap |

All triggers: third-party verified · binary · geofenced 5km · polled every 15 minutes.

---

## Persona

**Ravi Kumar, 27 — Multi-platform delivery worker, Kondapur, Hyderabad**

Active on Zepto + Swiggy. Earns ₹700–900/day. Loses ₹500–640 on flood days. No insurance. No savings buffer. Tier-1 city, flood-prone zone.

Target segment: Multi-platform Q-commerce + food delivery workers in Hyderabad, Mumbai, Chennai, Bengaluru.

---

## Premium Model

```
Weekly Premium = ₹35 (base) × Zone Risk Multiplier × Worker Risk Adjustment

Zone Risk Multiplier (0.8 → 1.4)
  Flood risk:   50% weight  (NDMA flood atlas)
  Curfew risk:  30% weight  (historical district data)
  Outage risk:  20% weight  (Downdetector historical)

Worker Risk Adjustment (0.85 → 1.15)
  Score > 0.75 → 0.85×  (discount — reward reliable workers)
  Score 0.50–0.75 → 1.00×
  Score < 0.50 → 1.15×  (surcharge)
```

| City | Base | Zone Adj | Worker Adj | Final |
|------|------|----------|------------|-------|
| Kondapur, Hyd (Low Risk) | ₹35 | ×1.40 | ×0.90 | ₹44 |
| Kurla, Mumbai | ₹35 | ×1.40 | ×1.00 | ₹49 |
| Koramangala, Blr | ₹35 | ×1.10 | ×1.00 | ₹39 |

---

## 5 AI / ML Modules

### 1 — Predictive Disruption Engine
Predicts floods before they happen. Judges love predictive intelligence — not just reactive automation.

- **Model:** XGBoost Regressor
- **Inputs:** IMD 7-day rainfall forecast, zone elevation (SRTM DEM), flood history (NDMA atlas), monsoon intensity index, soil moisture (NASA SMAP)
- **Output:** next_24h_disruption_probability per zone
- **Worker sees:** ⚠ High flood probability tomorrow · Coverage automatically extended
- **Admin sees:** Hyderabad Kondapur → 78% risk → 32 workers → ₹19,200 exposure

### 2 — Platform Activity Verification
Proves the worker was actually working before the trigger. Answers the judge question: *"How do you know they were working?"*

- **Source:** Mock delivery platform API (last_order_timestamp, GPS, active session logs)
- **Logic:** last_order_age < 2hrs → full payout · 2–4hrs → 75% · >6hrs → flagged
- **Output:** activity_verified: true/false · payout_multiplier: 0.75–1.0

### 3 — Worker Risk Score (Insurance Credit Score)
Behavior-based insurance pricing. Low-risk workers pay less. Real InsurTech practice applied to gig workers.

- **Score:** 0.0 → 1.0 (higher = lower risk = lower premium)
- **Components:** delivery_activity (35%), claim_frequency (30%), fraud_risk (20%), active_hours (15%)
- **Impact:** Score 0.82 → Low Risk → 0.85× multiplier → ₹5/week saved

### 4 — Multi-Worker Event Correlation
Collective signal to confirm disruptions and reduce fraud.

- **Logic:** >60% of zone workers claim same event → CONFIRMED → fraud scores reduced
- **Logic:** <10% or <3 workers → ANOMALY → flagged for review
- **Today:** Kondapur flood 28/33 = 84% → Auto-approved all · Bengaluru outage 1/14 = 7% → Flagged

### 5 — Fraud Detection Engine
7-check rule-based engine (Phase 2) → Isolation Forest (Phase 3).

- GPS distance · duplicate claim · claim frequency · IP city match · device consistency · activity verification · zone correlation ratio
- GPS spoofing detection: GPS inside zone + IP from different city = +0.80 fraud flag
- Score >0.70 → manual review · <0.70 → auto-approved → Razorpay payout

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TailwindCSS |
| Backend | FastAPI (Python 3.11) |
| Database | MongoDB (flexible document model for varied claim/fraud data) |
| Auth | Firebase OTP |
| ML — Premium | XGBoost |
| ML — Prediction | XGBoost Regressor |
| ML — Fraud | Isolation Forest (scikit-learn) |
| Flood Trigger | IMD SACHET RSS |
| Outage Trigger | Downdetector + platform status APIs |
| Activity Verify | Mock delivery platform API |
| Maps | Google Maps JS API |
| Payments | Razorpay test mode |
| Frontend Deploy | Vercel |
| Backend Deploy | Railway |

---

## MongoDB Schema

```javascript
// workers
{ phone, name, platforms[], zone:{lat,lng,district}, upi_id,
  risk_score, risk_tier, premium_multiplier,
  score_components:{delivery_activity, claim_frequency, fraud_risk, active_hours},
  last_order_ts, razorpay_fund_account_id }

// policies
{ worker_id, weekly_premium, coverage_cap,
  zone_multiplier, worker_multiplier, status, coverage_start, coverage_end }

// trigger_events
{ trigger_type, district, severity, source_url, fired_at,
  claims_count, zone_total_workers, correlation_ratio, confirmation_status }

// claims
{ worker_id, policy_id, trigger_event_id, trigger_type,
  claim_amount, activity_verified, last_order_age_minutes,
  fraud_score, fraud_flags[], correlation_boost,
  status, payout_reference, created_at, paid_at }

// disruption_forecasts
{ zone_district, prediction_date, disruption_probability, disruption_type,
  model_inputs:{imd_rainfall_mm, zone_elevation_m, flood_history_5yr, monsoon_index},
  estimated_affected_workers, estimated_payout_exposure }
```

---

## Automated Claim Flow

```
Every 15 minutes:
  Trigger engine polls IMD SACHET + Downdetector

On trigger detection:
  → Create TriggerEvent in MongoDB
  → Find all active workers in geofenced zone (5km)
  → For each worker:
      1. Check activity: last_order_timestamp
      2. Create claim if activity verified
      3. Run 7-check fraud engine
      4. Calculate zone correlation ratio
      5. fraud_score < 0.70 → auto-approve → Razorpay payout
      6. fraud_score > 0.70 → flag → admin queue
  → Push notification to worker
  → UPI payout within 2 hours
```

---

## Market Sizing

| Metric | Number |
|--------|--------|
| Gig delivery workers India | ~12 million |
| Tier-1 city workers (target) | ~4.2 million |
| TAM | ₹30,576 crore/year |
| SAM (4 cities, flood-prone) | ₹6,370 crore/year |
| SOM Year 1 (1.5% penetration) | ₹9.55 crore |
| Target loss ratio | 65% |

---

## Roadmap

**Phase 1 — Seed (Mar 4–20):** Architecture, prototype, all 5 AI module specs ✓

**Phase 2 — Scale (Mar 21–Apr 4):** Auth + onboarding, trigger engine, activity verification, correlation logic, rule-based fraud, mock payouts

**Phase 3 — Soar (Apr 5–17):** Predictive engine live, Isolation Forest fraud model, full Razorpay, complete dashboards, final pitch

---

## Team SentinelX — KL University

| | |
|--|--|
| Demo Video | [YouTube link] |
| Repository | github.com/[repo] |
| Prototype | [Vercel link] |

---

*Guide-Pay — We don't just react to disruptions. We predict them.*
#   G u i d e - P a y  
 