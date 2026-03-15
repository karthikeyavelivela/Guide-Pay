# Premium Model — Guide-Pay

## Formula

```
Weekly Premium = Base (₹49) × Zone Risk Multiplier × Worker Risk Adjustment
```

---

## Zone Risk Multiplier

**Range:** 0.8 → 1.4

Calculated from three risk components weighted by historical disruption frequency:

```
Zone Multiplier = 1 + (flood_score × 0.50) + (curfew_score × 0.30) + (outage_score × 0.20)

Where each score: 0.0 → 1.0
```

| Component | Weight | Data Source |
|-----------|--------|-------------|
| Flood risk | 50% | NDMA flood hazard atlas + IMD district rainfall records |
| Curfew risk | 30% | Historical Section 144 orders by district (2019–2024) |
| Outage risk | 20% | Downdetector historical incident data by city |

### City Multipliers

| District | Flood | Curfew | Outage | Multiplier |
|----------|-------|--------|--------|------------|
| Kondapur, Hyd | 0.85 | 0.25 | 0.40 | 1.40 |
| Kurla, Mumbai | 0.80 | 0.30 | 0.45 | 1.40 |
| Koramangala, Blr | 0.40 | 0.20 | 0.55 | 1.11 |
| T.Nagar, Chennai | 0.60 | 0.25 | 0.35 | 1.21 |
| Dwarka, Delhi | 0.25 | 0.65 | 0.30 | 0.96 |

---

## Worker Risk Adjustment

**Range:** 0.85 → 1.15

Calculated from the Worker Risk Score (Insurance Credit Score). Rewards reliable workers with lower premiums.

```
Score > 0.75 → multiplier = 0.85  (Low Risk — discount)
Score 0.50–0.75 → multiplier = 1.00  (Medium Risk — base rate)
Score < 0.50 → multiplier = 1.15  (High Risk — surcharge)
```

### Worker Risk Score Components

| Factor | Weight | Measurement |
|--------|--------|-------------|
| Delivery Activity | 35% | Orders per active day vs platform average |
| Claim Frequency | 30% | Claims per 100 active days vs zone average |
| Fraud Risk | 20% | Historical fraud flags + GPS consistency |
| Active Hours | 15% | Hours worked per week + peak coverage |

---

## Sample Calculations

| Worker | Zone | Risk Score | Zone Mult | Worker Mult | Premium |
|--------|------|------------|-----------|-------------|---------|
| Ravi Kumar | Kondapur, Hyd | 0.82 (Low) | 1.40 | 0.85 | **₹42** |
| Suresh M. | Kurla, Mumbai | 0.65 (Med) | 1.40 | 1.00 | **₹49** |
| Anita K. | Koramangala | 0.45 (High) | 1.11 | 1.15 | **₹45** |
| Rajesh D. | T.Nagar, Chennai | 0.78 (Low) | 1.21 | 0.85 | **₹36** |

---

## Coverage Caps by Trigger

| Trigger | Payout |
|---------|--------|
| IMD Flood Alert (Red/Orange) | 100% of weekly cap |
| Platform Outage (>2hrs) | 75% of weekly cap |
| Government Curfew | 100% of weekly cap |

**Weekly cap:** ₹600 (based on median daily earnings × 5 working days × 0.15 disruption factor)

---

## Actuarial Assumptions

| Parameter | Value | Basis |
|-----------|-------|-------|
| Expected loss ratio | 65% | Industry standard for parametric micro-insurance |
| Flood trigger frequency | 3.2 events/worker/year (flood zones) | IMD historical data |
| Outage trigger frequency | 1.8 events/worker/year | Downdetector data |
| Curfew trigger frequency | 0.4 events/worker/year | District records |
| Average payout per event | ₹520 | 87% of cap, activity verification adjustment |
| Fraud rate (pre-detection) | ~8% | Estimate from gig economy fraud literature |
| Fraud rate (post-detection) | <2% | With 7-check engine + correlation |

---

## Phase 3 — XGBoost Premium Model

Replace rule-based multipliers with a trained XGBoost Regressor:

```
Features (per worker-zone combination):
  - zone_flood_score, zone_curfew_score, zone_outage_score
  - worker_delivery_activity_score
  - worker_claim_frequency_score
  - worker_fraud_risk_score
  - worker_active_hours_score
  - season (monsoon/non-monsoon binary)
  - city_tier

Target:
  - expected_loss_ratio (calculated from historical payouts)

Output:
  - Optimal weekly premium that maintains 65% target loss ratio
```

Training data: synthetic (Phase 3) → real (post-launch)
