# Fraud Detection Engine — Guide-Pay

## Overview

Two-phase fraud system. Phase 2 uses rule-based scoring. Phase 3 upgrades to Isolation Forest with GPS spoofing detection.

---

## Phase 2 — Rule-Based Engine (7 Checks)

```python
def calculate_fraud_score(claim, worker, trigger_event):
    score = 0.0

    # Check 1 — GPS distance from zone (weight: 0.30)
    distance_km = haversine(claim.gps_lat, claim.gps_lng,
                            worker.zone.lat, worker.zone.lng)
    if distance_km > 10:
        score += 0.30
    elif distance_km > 5:
        score += 0.15

    # Check 2 — Duplicate claim for same event (weight: disqualify)
    if existing_claim_for_event(worker.id, trigger_event.id):
        return 1.0  # Instant disqualification

    # Check 3 — Claim frequency (weight: 0.20)
    recent_claims = count_claims_last_30_days(worker.id)
    if recent_claims > zone_average * 2.5:
        score += 0.20
    elif recent_claims > zone_average * 1.5:
        score += 0.10

    # Check 4 — IP city match (weight: 0.15)
    if claim.ip_city != worker.zone.city:
        score += 0.15

    # Check 5 — Device consistency (weight: 0.15)
    if claim.device_fingerprint != worker.registered_device:
        score += 0.15

    # Check 6 — Activity verification (weight: 0.25) ← NEW
    last_order_age = minutes_since(worker.last_order_ts)
    if last_order_age > 360:  # 6 hours
        score += 0.25
    elif last_order_age > 240:  # 4 hours
        score += 0.12

    # Check 7 — Zone correlation (weight: -0.30) ← FRAUD REDUCER
    ratio = trigger_event.claims_count / trigger_event.zone_total_workers
    if ratio > 0.60:
        score -= 0.30  # High correlation = event confirmed = lower fraud risk
    elif ratio < 0.10:
        score += 0.40  # Low correlation = anomaly = high fraud risk

    return max(0.0, min(1.0, score))
```

### Decision Threshold

```
score < 0.70 → Auto-approved → Razorpay payout initiated
score ≥ 0.70 → Flagged → Admin review queue
```

---

## Phase 3 — Isolation Forest

**Model:** sklearn.ensemble.IsolationForest

**Training Data:**
- 800 clean claims (synthetic, based on verified trigger events)
- 40 fraud samples (synthetic fraud patterns):
  - GPS far from zone + claim filed
  - Zero activity in 6h before trigger
  - IP city mismatch
  - Solo claimer in a zone with 20+ workers
  - Device switch 24h before claim

**Features:**
```
[gps_distance_km, claim_frequency_30d, last_order_age_min,
 ip_city_match, device_consistent, zone_correlation_ratio,
 worker_risk_score, hour_of_claim, day_of_week]
```

**Output:** anomaly_score → mapped to 0.0–1.0 fraud probability

---

## GPS Spoofing Detection

Condition: GPS inside zone + IP geolocation in a different city

```python
def detect_gps_spoofing(claim):
    ip_city = geoip_lookup(claim.ip_address)
    gps_city = reverse_geocode(claim.gps_lat, claim.gps_lng)
    
    if ip_city != gps_city:
        claim.fraud_flags.append("GPS_SPOOFING_SUSPECTED")
        return 0.80  # Add 0.80 to fraud score
    return 0.0
```

---

## Multi-Worker Event Correlation

The most important fraud signal. Individual fraud is hard to fake collective behavior.

```python
def calculate_correlation_boost(trigger_event_id, worker_zone):
    event = TriggerEvent.find(trigger_event_id)
    ratio = event.claims_count / event.zone_total_workers
    
    if ratio >= 0.60:
        # Event confirmed by collective behavior
        # Reduce all fraud scores for this event
        apply_fraud_reduction(trigger_event_id, reduction=0.30)
        event.confirmation_status = "CONFIRMED"
    
    elif ratio <= 0.10 and event.claims_count < 3:
        # Anomaly — too few workers claimed
        # Flag all single claimers
        flag_solo_claimers(trigger_event_id, additional_score=0.40)
        event.confirmation_status = "ANOMALY"
```

---

## Activity Verification Logic

Before any claim is processed, Guide-Pay checks the worker was actively working.

```python
def verify_activity(worker_id, trigger_timestamp):
    worker = Worker.find(worker_id)
    last_order_age = (trigger_timestamp - worker.last_order_ts).total_minutes()
    
    if last_order_age < 120:   # Active in last 2 hours
        return {"verified": True, "multiplier": 1.0}
    
    elif last_order_age < 240:  # Active in last 4 hours
        return {"verified": True, "multiplier": 0.75}
    
    elif last_order_age < 360:  # Active in last 6 hours
        return {"verified": "partial", "multiplier": 0.50}
    
    else:  # No activity in 6+ hours before trigger
        return {"verified": False, "multiplier": 0.0, "flag": "NO_ACTIVITY"}
```

---

## Admin Review Interface

Claims flagged (score ≥ 0.70) enter an admin queue with:

| Field | Display |
|-------|---------|
| Worker name + ID | Quick identification |
| Fraud score | 0.0–1.0 with colour coding |
| Triggered flags | Which of the 7 checks failed |
| GPS map | Worker location vs zone boundary |
| Activity timeline | Last 24h order history |
| Zone correlation | How many other workers claimed same event |
| Action | Approve / Reject / Request verification |

Admin target: resolve flagged claims within 4 hours.
