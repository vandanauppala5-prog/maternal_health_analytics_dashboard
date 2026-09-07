# Maternal Health Analytics — Documentation

## 1. Data sources & assumptions

**What's real vs. modeled — read this first.** This project uses a
**realistic synthetic dataset**, as explicitly permitted by the task brief
("...or a realistic anonymized/synthetic dataset"). Real, row-level app/tech
engagement data for maternal health platforms is not publicly published
anywhere (it's proprietary to individual platforms), so a synthetic
patient-level table is the only workable option for this kind of brief. The
goal was to make it *defensible*, not decorative — every distribution is
anchored to a real, citable pattern rather than picked arbitrarily.

| Element | Basis | Status |
|---|---|---|
| **Tamil Nadu MMR = 25 per 100,000 live births (2022-24)** | **SRS Special Bulletin on Maternal Mortality in India, 2022-24** — Office of the Registrar General, India (ORGI), Sample Registration System, https://censusindia.gov.in/. PDF obtained and verified directly (`sources/srs_bulletin_extract.md` has the extracted table). 95% CI: 3–46. | **REAL, cited figure** |
| **India MMR trend, 2014-16 through 2022-24 (130 → 87)** | Same SRS bulletin, Section 4 / Figure 1 — published year-by-year at the national level | **REAL, cited** |
| District-year MMR is NOT published by any official source | SRS explicitly pools 3 years of data even at *state* level because maternal death is a rare event requiring a large sample — district-level estimates would be statistically unreliable, so they simply aren't produced | N/A — no real data exists at this grain |
| Tamil Nadu district-level MMR split (15 districts) | **Modeled**: the real state anchor (25) distributed across districts using a risk multiplier informed by real, documented urban/rural institutional-delivery differentials. The district-model's live-birth-weighted average lands at 26.1, close to the real 25 anchor — used as an internal consistency check (see `MMR_Trend` tab in the Excel workbook) | Modeled, disclosed via the `mmr_basis` column on every row |
| App reminders, telehealth consults, content views, risk flags | Fully synthetic, generated with documented probability rules (see `generate_data.py`) that skew tech engagement toward urban/younger patients and away from higher-risk districts, matching real-world patterns reported in maternal-health digital-health literature | Synthetic |

**What changed from an earlier draft:** an initial pass at this dataset used
a *recalled* (not verified) Tamil Nadu MMR estimate of ~54–58. After
obtaining and reading the actual SRS bulletin, the real figure is
**25** — less than half the earlier estimate. All tables, SQL, the Excel
workbook, and the dashboard were rebuilt around the verified number. This is
flagged here deliberately, since catching and correcting a wrong assumption
before submission is part of the analysis, not something to hide.

## 2. Data cleaning steps (see `sql/maternal_health_analysis.sql` §1)
1. **Duplicates** — exact duplicate `patient_id` rows removed (kept first
   occurrence); duplicate district-year rows in the regional table removed.
2. **Inconsistent categorical entries** — `region_type` values (`urban`,
   `Urban`, `RURAL`, `Rural`) standardized to `Urban`/`Rural`; district name
   casing standardized.
3. **Missing values** — missing `prenatal_checkups` imputed with the
   district + region_type average (not a global average, since checkup
   access genuinely varies by district and urban/rural setting); missing
   `live_births` imputed with that district's average across other years.
   Rows missing `district` or `age_group` (identifying fields) were dropped
   rather than imputed.
4. **Implausible values** — `prenatal_checkups > 14` treated as a data-entry
   error, nulled, then imputed as above (14+ checkups in a single pregnancy
   is outside any realistic ANC schedule).

## 3. Key insights (see Insight_Summary for the stakeholder-facing version)
- Tamil Nadu's real MMR (25 per 100k, 2022-24) is dramatically better than
  the national figure (87) and one of the lowest of any Indian state —
  the state overall is a public-health success story, not a crisis area.
- Even so, the modeled district split shows meaningful *internal* variation
  (roughly 16 to 33 per 100k across districts) — several rural districts
  (Ramanathapuram, Nagapattinam, Villupuram) sit well above the state
  average even though all are low by national standards.
- Urban patients show a higher institutional-delivery rate, more prenatal
  checkups, and higher tech engagement than rural patients.
- Tech engagement (app reminders + telehealth + content) correlates
  positively with prenatal checkup completion (r ≈ 0.40) — meaningful but
  moderate, meaning tech adoption is *associated with* better engagement,
  not a guaranteed cause.
- The districts with the *worst* combination of (relatively) high MMR and
  *low* tech adoption are the clearest outreach targets — see the priority
  table. Framing matters here: this is about narrowing an already-small gap
  within a strong-performing state, not "fixing a crisis."

## 4. Recommendations
1. **Target the 4–5 highest-priority districts first** (Ramanathapuram,
   Nagapattinam, Villupuram, Virudhunagar, Cuddalore) with a combined
   outreach + tech-adoption push — they show the worst MMR/tech-adoption
   combination.
2. **Push telehealth harder in the 1st trimester** — reminder usage and
   checkup completion are both lowest in trimester 1, the highest-leverage
   window for early risk detection.
3. **Rural-specific onboarding flow** — rural patients trail urban patients
   on every tech metric; a lighter-weight (SMS-based reminder, not just
   in-app) channel would likely close more of that gap than promoting the
   same app experience harder.
4. **Flag high-risk + low-engagement patients for proactive telehealth
   outreach** rather than waiting for them to self-initiate — this segment
   is small but carries disproportionate risk.
5. **Re-validate this model against real SRS/NFHS-5 figures** before using
   it for actual budget or staffing decisions (see Limitations).

## 5. Limitations
- **District-level MMR is modeled, not measured** — real SRS data does not
  go below state level (confirmed directly from the bulletin itself), so
  treat district MMR as a *directional estimate*, not a precise figure. The
  state-level anchor (25) is real and verified.
- **Tamil Nadu's MMR has a wide confidence interval (3–46)** in the source
  bulletin, reflecting how few maternal deaths a state sample captures even
  pooled over three years — the point estimate of 25 should be treated with
  that uncertainty in mind, not as a precise count.
- **No real year-by-year Tamil Nadu series exists** — only India's national
  MMR is published annually; the state figure is a single 2022-24 pooled
  estimate. The dashboard's trend chart is therefore the real *national*
  trend, with Tamil Nadu shown as a single reference point, not a fabricated
  state trend line.
- **Tech-usage data is entirely synthetic** — the correlation and
  segmentation findings show what a *plausible* pattern looks like, useful
  for demonstrating the analysis approach, but should not be quoted as an
  actual measured outcome of any real platform.
- **Correlation ≠ causation** — the tech-engagement/checkup correlation
  (r≈0.40) doesn't establish that using the app *causes* more checkups;
  more-engaged patients may simply be more health-conscious on both fronts.
- **.twbx (Tableau) not produced** — the dashboard deliverable is a native Power BI .pbix file, built directly in Power BI Desktop from the cleaned     CSVs and the DAX measures documented in this project.
## 6. File index
```
data/
  regional_mmr_raw.csv, patient_records_raw.csv      - raw, with intentional data-quality issues
  regional_mmr_clean.csv, patient_records_clean.csv  - cleaned tables (see SQL §1)
  india_mmr_trend_real.csv                            - REAL national MMR trend, 2014-16 to 2022-24
  outreach_priority.csv                               - priority ranking output
  Maternal_Health_Supporting_Analysis.xlsx             - Excel supporting analysis (formulas, not hardcoded)
sql/
  maternal_health_analysis.sql                         - full SQL: cleaning, MMR analysis, tech analysis, priority view
dashboard/
  Maternal_Health_Dashboard.pbix                       - interactive 1-page dashboard (filters + 6 views)
docs/
  README.md (this file)
  Insight_Summary.md                                    - 200-300 word stakeholder summary
sources/
  srs_bulletin_extract.md                               - extracted real figures + citation from the SRS PDF
  SRS_MMR_Bulletin_2022_2024.pdf                         - the source document itself
```
