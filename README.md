# A/B Testing: Landing Page Conversion Analysis

A complete A/B test analysis in Python evaluating whether a new landing page (treatment) outperforms the existing page (control) on conversion rate and purchase revenue, with rigorous integrity checks and segment-level validation.

## Overview

This project analyzes a 294,478-row experiment dataset to determine whether a landing page redesign should be shipped. Rather than stopping at a single significance test, the analysis validates the experiment's integrity first (group/page mismatches, duplicate users, conversion logic consistency), checks whether the study was adequately powered, then runs primary and secondary hypothesis tests with confidence intervals and effect sizes, projects revenue impact, and stress-tests the result across segments to rule out Simpson's Paradox before making a ship/no-ship recommendation.

## Tech Stack

- **Language:** Python
- **Libraries:** pandas, numpy, scipy, statsmodels (proportions_ztest, NormalIndPower, proportion_effectsize)

## Methodology

The analysis follows nine sections:

1. **Load & Inspect** — Shape, null counts, descriptive stats, group sizes, and demographic balance (gender, age) across control/treatment to confirm randomization worked as expected.
2. **Data Quality & Integrity Checks** — Detects and removes rows where group assignment doesn't match landing page (e.g., control users who saw the new page), removes duplicate users, and checks for logical inconsistencies between `converted` and `purchase_amount`.
3. **Sample Size & Statistical Power** — Computes Cohen's h for the observed effect and solves for the minimum sample size needed to detect it at α=0.05, power=0.80, then compares against actual group sizes to confirm the study is adequately powered before trusting the p-value.
4. **Primary Test — Conversion Rate** — Two-proportion Z-test comparing control vs. treatment conversion rates, with a 95% confidence interval on the absolute difference and relative lift.
5. **Secondary Test — Purchase Amount** — Welch's t-test (unequal variance) on purchase amount, with Cohen's d effect size, a 95% CI on the mean difference, and a repeat of the test restricted to converters only (a more meaningful lens on spend behavior).
6. **Revenue Impact Projection** — Extrapolates observed ARPU (average revenue per user) in each group to monthly and annual uplift if the treatment were rolled out to 100% of traffic.
7. **Segment Analysis** — Re-runs the conversion Z-test within device type, gender, location, and age group to check whether the effect holds consistently or is concentrated in a subset of users.
8. **Simpson's Paradox Check** — Scans every segment for cases where treatment underperforms control despite an overall positive result — a critical check before recommending a full rollout.
9. **Final Summary** — Consolidates all tests into a single decision block: hypothesis, primary/secondary metric results, revenue impact, study health, and a final ship/no-ship recommendation.

## Key Design Decisions

- **Integrity checks run before any hypothesis test.** Group/page mismatches and duplicate users are removed first, since testing on contaminated groups would invalidate every result downstream.
- **Power analysis precedes the significance test**, not after — confirming adequate sample size before interpreting a p-value guards against both underpowered false negatives and over-trusting a marginally significant result.
- **Effect size accompanies every p-value** (Cohen's h for proportions, Cohen's d for the continuous purchase-amount metric), since statistical significance alone doesn't indicate practical significance at scale.
- **Converters-only purchase comparison** is run alongside the full-population comparison, since average purchase amount across all users conflates "did they buy" with "how much did they spend given they bought."
- **Segment analysis and Simpson's Paradox check are mandatory gates**, not optional extras — an aggregate lift can mask a segment where the change actually hurts conversion, which the paradox check is specifically designed to catch before a full rollout.

## How to Run

1. Install dependencies: `pip install pandas numpy scipy statsmodels`
2. Update the file path in Section 1 to point to your local copy of `abt.csv`.
3. Run the script top to bottom — each section builds on cleaned data and variables from the previous one (e.g., Section 4's `n_control`/`n_treatment` feed Section 3's power calculation).
4. Console output prints each section's results in sequence, ending with a consolidated summary block and final recommendation.

## Notes

- Revenue projections in Section 6 are explicitly framed as directional estimates, assuming the study's observed traffic volume approximates typical monthly traffic — this assumption is stated in the code and should be caveated when presenting results.
- The age group segmentation (`18–24`, `25–34`, `35–44`, `45–65`) is defined via `pd.cut` and reused across the segment and Simpson's Paradox checks for consistency.
