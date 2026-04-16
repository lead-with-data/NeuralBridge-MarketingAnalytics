<img width="2400" height="1784" alt="Gemini_Generated_Image_5nc8ia5nc8ia5nc8" src="https://github.com/user-attachments/assets/5af58446-bf99-4ee1-9d89-09b34dac4c06" />


# NeuralBridge AI — Marketing Performance Analysis

## Problem

NeuralBridge AI is a Series B B2B SaaS company selling an AI infrastructure platform to enterprise engineering teams. At the end of FY2024, the company had spent its full $1.74M marketing budget and closed $2.88M in ARR against an $18M target — a gap of $15.1M with no qualified pipeline remaining to close it.

The core question was not just why the number was missed, but which channels were responsible, whether the budget was allocated correctly, and what to do in Q1 2025 to course-correct.

---

## Data

Six raw source files were used, each exported from a different system. None were pre-cleaned or joined. All preprocessing was done before analysis.

| File | Source | Rows | Key Issues |
|---|---|---|---|
| `linkedin_ads_export.csv` | LinkedIn Campaign Manager | 1,098 | Dates in MM/DD/YYYY, 3–5 day spend lag, CPM rounding discrepancy |
| `google_ads_export.csv` | Google Ads UI | 3,660 | CTR is a string percent, Quality Score null on PMax, Impression Share is "--" |
| `hubspot_contacts_export.csv` | HubSpot CRM | 2,800 | Lead Score scale changed Feb 14 (0–50 → 0–100), ~8% missing campaign ID, ~4% duplicate emails |
| `hubspot_deals_export.csv` | HubSpot CRM | 357 | Deal Stage has 6 naming variants, ~4% closed-won with no close date, ~3% with $0 value |
| `webinar_platform_export.csv` | Zoom / ON24 | 3,468 | Attendance Duration = 0 for both no-shows and <1 min joins, 34% no CRM match |
| `content_syndication_leads.csv` | TechTarget / Bombora / IDG / Foundry | 520 | 77.6% already exist in HubSpot, column names differ from CRM, BANT fields 40% Unknown |

Finance actuals were taken from `finance_budget_actuals.xlsx` (two tabs: Budget\_Actuals and Vendor\_Contracts) as the authoritative spend source. Platform-reported spend was not used for CAC calculations because LinkedIn invoiced 3–8% above what the platform reported.

---

## Table Relationships

```
hubspot_contacts_export
        │
        │  Contact ID = Associated Contact ID
        ▼
hubspot_deals_export ◄──── attribution_touches (JSON, post-April 2024 only)
        │
        │  Original Source Channel → Campaign Name / Campaign
        ▼
linkedin_ads_export  /  google_ads_export  (joined on channel + month)
        │
        │  HubSpot Contact ID = Contact ID
        ▼
webinar_platform_export
        │
        │  Email = Email  (after dedup)
        ▼
content_syndication_leads
        │
        │  Channel Spend (authoritative)
        ▼
finance_budget_actuals.xlsx → Budget_Actuals tab
```

The master funnel table was built by joining contacts to deals. All channel spend was reconciled against Finance actuals before computing CAC.

---

## What Was Built

- Preprocessed silver layer saved to Databricks Volumes as CSV (6 tables + master funnel)
- PySpark analysis covering all channel, funnel, and revenue questions
- HTML dashboard with 8 pages, filters by quarter / segment / channel / scoring version / vendor

---

## Key Insights

### Revenue

| Metric | Value |
|---|---|
| Closed ARR | $2.88M |
| Target | $18M |
| Gap | $15.1M |
| Pipeline coverage | 0.0x |
| Total customers won | 57 |

### CAC and LTV:CAC by Channel

| Channel | Spend | Customers | CAC | Avg LTV | LTV:CAC |
|---|---|---|---|---|---|
| Webinar | $313,200 | 27 | $11,600 | $60,921 | 5.25x |
| Google Search | $382,800 | 19 | $25,520 | $56,515 | 2.81x |
| Email Nurture | $174,000 | 3 | $58,000 | $66,752 | 1.15x |
| SEO | $139,200 | 2 | $69,600 | $42,826 | 0.62x |
| Content Syndication | $243,600 | 3 | $81,200 | $47,028 | 0.58x |
| LinkedIn | $487,200 | 3 | $162,400 | $46,316 | 0.29x |

### Funnel Conversion by Channel

| Channel | MQLs | SQLs | MQL→SQL Rate |
|---|---|---|---|
| Google Search | 270 | 104 | 38.5% |
| Webinar | 271 | 100 | 36.9% |
| SEO | 54 | 17 | 31.5% |
| LinkedIn | 237 | 46 | 19.4% |
| Email Nurture | 59 | 11 | 18.6% |
| Content Syndication | 137 | 12 | 8.8% |

Benchmark for MQL→SQL: 28%. Content Syndication is 19 points below benchmark.

### Scoring Model Impact (August 3 change)

| Version | LinkedIn MQLs | LinkedIn SQL Rate |
|---|---|---|
| v1 (before Feb 14) | 23 | 17.4% |
| v2 (Feb 14 – Aug 3) | 108 | 21.3% |
| v3 (after Aug 3) | 106 | 17.9% |

LinkedIn's problem is SQL conversion rate, not MQL volume. MQL volume held between v2 and v3 but conversion dropped.

### Content Syndication Quality

| Metric | Value |
|---|---|
| Total opted-in leads | 500 |
| Already in HubSpot | 388 (77.6%) |
| Net new leads | 112 |
| SQL conversion rate | 8.76% |
| Benchmark | 28% |
| Gap | -19.2% |

### Google Ads

| Campaign Type | Cost/Conversion | Spend |
|---|---|---|
| Search | $5.15 | $323,806 |
| Performance Max | $6.93 | $34,822 |

Best days by conversions: Tuesday (10,785), Thursday (10,629), Monday (10,559).

### Webinar

| Metric | Value |
|---|---|
| Total registrants | 3,468 |
| Genuine attendees (>5 min) | 1,831 |
| MQLs within 30 days | 338 |
| MQL-to-SQL rate | 36.9% |
| Customers won | 24 |
| Closed ARR | $1.41M |
| CAC | $13,050 |
| ARR / spend ratio | 4.49x |

### Deal Stage Drop-off

| Stage | Count | Share |
|---|---|---|
| Closed Lost | 159 | 44% |
| Stalled at SQL | 140 | 39% |
| Closed Won | 58 | 16% |

Most deals never reach Opportunity stage. The SQL-to-close conversion is the primary bottleneck.

---

## Recommendations

**1. Cut LinkedIn by 50%**
LinkedIn spent $487,200 to win 3 customers at a CAC of $162,400 and LTV:CAC of 0.29x. Every dollar spent returns 29 cents of lifetime value. Reduce spend to $243,600 in Q1 2025 and reallocate the freed budget.

**2. Pause Content Syndication**
77.6% of vendor-delivered leads already exist in HubSpot. Net new leads are 112 against $243,600 in spend. SQL conversion is 8.76% vs 28% benchmark. No vendor meets minimum performance threshold. Pause all contracts in Q1 2025.

**3. Scale Webinar**
Webinar is the only channel with LTV:CAC above 3x (5.25x). CAC is $11,600 — 14x lower than LinkedIn. Add $300,000 in Webinar budget using freed spend from LinkedIn and Syndication cuts. Projected additional customers: ~23.

**4. Shift Google PMax budget to Search**
Search delivers conversions at $5.15 vs $6.93 for Performance Max — a 34% difference. Reallocate the $34,822 PMax budget to Search campaigns.

**5. Fix the lead scoring model**
The August 3 threshold change dropped LinkedIn MQL-to-SQL from 21.3% to 17.9%. Review the scoring threshold and recalibrate to avoid disqualifying leads that sales would accept.

---

## Files

```
/
├── generate_data.py                        synthetic data generator
├── linkedin_ads_export.csv                 raw LinkedIn export
├── google_ads_export.csv                   raw Google Ads export
├── hubspot_contacts_export.csv             raw HubSpot contacts
├── hubspot_deals_export.csv                raw HubSpot deals
├── webinar_platform_export.csv             raw webinar registrants
├── content_syndication_leads.csv           raw vendor leads
├── finance_budget_actuals.xlsx             authoritative spend (Finance)
└── NeuralBridge_AI_Raw_Data_Files.zip      all raw files packaged
```

Silver layer (preprocessed, Databricks Volumes):
```
/Volumes/neuralbridge-marketing/default/neuralbridge-marketing-silver/
├── google_ads/
├── linkedin_ads/
├── hubspot_contacts/
├── hubspot_deals/
├── webinar_platform/
├── content_syndication/
└── master_funnel/
```
