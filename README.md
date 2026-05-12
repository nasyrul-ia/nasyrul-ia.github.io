# IA System Monitoring

> Automated anomaly detection across the affiliate network — flagging high sale amounts, offer-level CVR drops, and publisher-level anomalies before they cause financial exposure.

**Status:** Pending Approval | **Data Source:** app.involve.asia | **Output:** Google Sheets (Daily)

---

## Overview

As the affiliate network grows in volume, manual oversight of conversion data is no longer sustainable. This system introduces **three automated monitoring modules** that run daily, surface anomalies instantly, and publish findings to a shared Google Sheet — so the team spends time fixing issues, not finding them.

---

## Modules

### Module 1 — High Sale Amount Monitoring

Flags any Pending or Yet-to-Consume conversion exceeding **MYR 200K**. If the offer has approved records ≥ MYR 200K in the last 90 days, a **1.5× dynamic threshold** is also applied as an additional check.

**Flagging Rule (either condition triggers a flag):**

1. **Hard Floor:** Any Pending/YTC conversion with Sale Amount > MYR 200,000 is always flagged.
2. **Dynamic Threshold:** If the offer's max approved amount in the last 90 days ≥ MYR 200K, also apply `Max Approved × 1.5` — flag any conversion exceeding that value.

**Detection Steps:**

1. Establish baseline — pull all Approved conversions from last 90 days per offer, find max sale amount.
2. Apply Hard Floor — flag any Pending/YTC conversion > MYR 200K.
3. Apply Dynamic Threshold (when applicable) — flag conversions exceeding `Max Approved × 1.5`.
4. Auto-Archive — snapshot flagged conversion values (At Detection vs. Latest) so edits are fully visible.

**Google Sheets Tabs:**
- **HSA Summary** — all offers with max approved amount, dynamic threshold, and current max (Pending + YTC).
- **HSA Conversions** — flagged conversion records (Conv ID, Sale Amt, Revenue, Payout).
- **HSA Archive** — at-detection vs. latest values for each flagged conversion.

---

### Module 2 — Offer CVR Monitoring

Tracks click-to-conversion rate per offer against a **30-day baseline**, alerting at three drop thresholds — with active and affected publisher count per flagged offer.

**Formula:** `CVR = Total Conversions ÷ Total Clicks` (per Offer, per Day — 1 Day Lag)

**Dual-Metric Status (Option C):**

| Metric | Calculation | Purpose |
|--------|-------------|---------|
| **Status (vs 30D)** | Current CVR ÷ 30-day rolling average | Primary long-term health indicator |
| **Status (vs 7D)** | Current CVR ÷ 7-day rolling average | Early-warning layer — catches sudden drops or recoveries before they show in the 30D baseline |

**Tier Classification:**

| Tier | Threshold | Meaning |
|------|-----------|---------|
| Normal | 100% | Within expected range |
| Concern | ≥90% | Slight dip — monitor |
| Warning | ≥70% | Significant drop — investigate |
| Critical | ≥50% | Severe — immediate action |

> When the two statuses diverge: 30D=Normal + 7D=Warning signals a sudden drop; 30D=Warning + 7D=Normal suggests CVR is stabilising.

**Detection Steps:**

1. Calculate 30-day average daily CVR per offer (the Normal/100% reference).
2. Calculate 7-day average CVR as an early signal.
3. Classify current CVR against both baselines independently.
4. Count active and affected publishers per offer. If affected ratio < 90%, refer to Module 3 for publisher drill-down.

**Google Sheets Tabs:**
- **CVR Summary** — full offer detail with 30D thresholds, 7D avg, current CVR, and dual status.
- **CVR Affected Offers** — case records with remarks for each flagged offer.

---

### Module 3 — Publisher CVR Monitoring

Drill-down into individual publisher CVR when Module 2 flags an offer. The affected-to-active publisher ratio classifies the issue and directs the right escalation path.

> This module only activates when an offer reaches **Concern tier or above** in Module 2.

**Affected Ratio — Triage Diagnostic:**

| Affected / Active | Classification | Signal | Recommended Action |
|-------------------|---------------|--------|--------------------|
| ≥90% | Systemic | All/nearly all publishers affected — offer-side or advertiser-side problem | AM escalates to advertiser immediately |
| 50–89% | Mixed | Majority affected — possible offer issue or shared traffic patterns | Check patterns; investigate both offer and publisher sides |
| 10–49% | Isolated | Minority affected — likely publisher-specific | Integration Team contacts specific affected publishers |
| 1 publisher | Single Point | Single publisher driving the drop | Deep-dive that publisher; consider temporary pause |

**Detection Steps:**

1. Triggered by Module 2 flag — auto-generates publisher-level CVR breakdown.
2. Calculate each publisher's own 30D and 7D average CVR (independent baseline).
3. Apply same dual-metric status (vs 30D / vs 7D) with same four-tier classification.
4. Calculate affected ratio and classify (Systemic / Mixed / Isolated / Single Point).

**Google Sheets Tabs:**
- **PUB CVR Publisher Detail** — per-publisher thresholds, status, and affected flag.
- **PUB CVR Affected Publishers** — case log with classification and remarks.

---

## Master Tab

A single Google Sheet tab provides a **network-wide view at a glance** — one row per offer, with High Sale Amount and CVR status side by side. Flagged cells are highlighted so issues are immediately visible without switching tabs.

| Offer ID | Offer Name | Status | HSA: Max Approved | HSA: Dynamic Threshold | HSA: Current Max (P+YTC) | CVR: 30D Avg | CVR: 7D Avg | CVR: Current | Status (30D) | Status (7D) |
|----------|-----------|--------|-------------------|----------------------|--------------------------|--------------|-------------|-------------|---------------|-------------|

---

## SOP & Process Flow

### Module 1 — High Sale Amount: Response

| Step | Action | Owner |
|------|--------|-------|
| 1 | System detects flag & logs conversion snapshot to archive | System |
| 2 | Notify IA Ops + Business Unit — total amount & conversion details | System |
| 3 | System zeroes sale amount — flagged conversion IDs written to Google Sheets, status stays Pending | System |
| 4 | Monitor for re-flag / close case | System + IA Ops |

### Module 2 — CVR Monitoring: Response

| Step | Action | Owner |
|------|--------|-------|
| 1 | System detects tier breach & logs flag | System |
| 2 | Notify Integration Team | System |
| 3 | Integration Team fast scan — triage root cause | Integration Team |
| 3a | ↳ False alarm (seasonality/pause) — document & close | Integration Team |
| 3b | ↳ Advertiser issue — AM escalates & communicates with advertiser | Account Manager |
| 3c | ↳ Internal issue — Tech Team troubleshoots & fixes | Tech Team |
| 3d | ↳ Publisher-level issue (affected ratio <90%) — open Module 3 report | → Module 3 |
| 4 | Provide ETA to stakeholders **(mandatory)** | Integration / AM |
| 5 | Monitor — verify metrics return to Normal tier | System + Integration |
| 6 | Notify all stakeholders of resolution — close case | Integration / AM |

### Module 3 — Publisher CVR: Response

| Step | Action | Owner |
|------|--------|-------|
| 1 | Module 2 flags an offer — Module 3 publisher report auto-generated | System |
| 2 | Integration Team reviews publisher report & assigns classification | Integration Team |
| 2a | ↳ Systemic (≥90%) — AM escalates to advertiser | Account Manager |
| 2b | ↳ Mixed (50–89%) — check common patterns | Integration Team |
| 2c | ↳ Isolated/Single (<50%) — contact specific publishers | Integration Team |
| 3 | Provide ETA to stakeholders — monitor until publishers return to Normal | Integration / AM |
| 4 | Close case — update Module 2 log with publisher-level resolution | Integration Team |

---

## Offer Health Report

A monthly flag count per offer, broken down by module — **HSA** and **CVR**. The Offer Health team picks up this data for their own evaluation and scoring. Our responsibility is to provide accurate, consistent counts.

| Offer ID | Offer Name | Month | HSA Flags | CVR Flags |
|----------|-----------|-------|-----------|-----------|

---

## Tech Stack

- **Data Source:** app.involve.asia (Involve Asia platform)
- **Output:** Google Sheets (updated daily)
- **Monitoring:** Automated daily runs with tier-based alerting

---