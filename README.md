## Pipeline Health Check — EDA, Corruption & Distribution Shift

**Objective:** Diagnosed and remediated five distinct data-quality corruptions in a synthetic country-panel dataset using pure exploratory data analysis, then quantified feature-level distribution shift between training and inference splits using the Population Stability Index (PSI) to validate a production monitoring approach.

**Methodology:**
- Performed manual EDA (summary statistics, histograms, duplicate-key checks) to surface five planted corruptions: negative GDP values, life expectancy recorded in months instead of years, near-duplicate country-year rows, inconsistent percentage units (decimal vs. whole-number) mixed with out-of-range values in `trade_pct`, and a GDP unit mismatch (values scaled down by a factor of ~1,000)
- Diagnosed each corruption using the appropriate technique — descriptive statistics for magnitude errors, log-scale histograms for multiplicative/ratio-scale bugs invisible on linear axes, and key-uniqueness checks for near-duplicates that full-row deduplication misses
- Benchmarked manual EDA against automated profiling (ydata-profiling) to identify which corruptions each approach catches independently, and which require domain-specific plausibility rules no generic profiler encodes
- Computed the Population Stability Index between training and inference splits to detect distribution shift, with GDP PSI = 2.4589 (flagged as a significant shift), while validating that conventional 0.10/0.25 PSI thresholds are heuristics without an attached null distribution — not statistical tests — and are sensitive to sample size
- Authored `eda_utils.py`, a reusable module exposing `check_impossible_values()` (constraint-based validation), `detect_distribution_shift()` (PSI computation), and `eda_summary()` (diagnostic reporting)
- Built an interactive, tabbed dashboard (ipywidgets + Plotly) combining a train/inference distribution overlay, a PSI threshold explorer, editable constraint validation, and a before/after remediation comparison

**Key Findings:**
- All five planted corruptions were identified and corrected without automated tooling; automated profiling independently surfaced 3 of 5 (negative GDP, the life-expectancy outlier, and the `trade_pct` range anomaly) but missed the duplicate rows and the GDP unit mismatch, both of which required domain-specific reasoning (key uniqueness and log-scale visualization, respectively) that a generic profiler doesn't encode
- The deliberately shifted feature (GDP) registered a PSI of 2.4589, well above the conventional "significant shift" threshold, while unshifted features returned PSI values in the 0.09–0.23 range purely from small-sample binning noise — underscoring that fixed PSI thresholds require calibration against a bootstrapped null distribution rather than textbook conventions, particularly at low sample sizes
- Constraint-based validation flagged 104 rows in the dirty dataset (0 in the cleaned dataset) under bounds derived from the cleaned data's observed range, confirming the remediation fully resolved the encoded corruptions
