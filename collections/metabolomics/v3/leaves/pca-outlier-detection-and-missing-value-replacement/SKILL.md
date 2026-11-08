---
name: pca-outlier-detection-and-missing-value-replacement
description: Use when curating or reproducing QC filtering of GCxGC-TOFMS plasma metabolite
  peak tables — specifically when deciding which detected peaks to exclude (run-order
  drift, QC deviation, PCA outliers) and which metabolites need missing-value replacement,
  as in the 691→195 peak filter.
when_to_use_negative:
- The input is already a cleaned, retained feature table (e.g., the 195-metabolite
  matrix used for OPLS-DA modeling) — filtering has already been performed.
- The data are raw GCxGC-TOFMS instrument files that have not yet been peak-picked;
  run peak detection and annotation first before applying these QC thresholds.
- The dataset lacks QC samples or run-order information, so the Pearson |r|>0.5 and
  QC RSD 0.3–0.5/0.5 criteria cannot be evaluated.
tools:
- name: GCxGC-TOFMS
  role: Instrument platform that generated the 691-peak plasma metabolite dataset
    to which the PCA outlier (value > 4, replacement > 5) and QC-RSD filtering criteria
    were applied
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:506c5cc8a6e6fcc84b6060ff94949826a431018d32915b8b9e15f50531aca98e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pca-outlier-detection-and-missing-value-replacement

## Summary

Audit and apply the GCxGC-TOFMS peak-QC scheme from the typhoid carriage metabolomics study, in which PCA (centered/UV-scaled) outlier metabolites are flagged at value > 4 and their missing values replaced at > 5, alongside run-order correlation and QC-RSD exclusions that reduced 691 detected peaks to 195 retained metabolites.

## When to use

Use when curating or reproducing QC filtering of GCxGC-TOFMS plasma metabolite peak tables — specifically when deciding which detected peaks to exclude (run-order drift, QC deviation, PCA outliers) and which metabolites need missing-value replacement, as in the 691→195 peak filter.

## When NOT to use

- The input is already a cleaned, retained feature table (e.g., the 195-metabolite matrix used for OPLS-DA modeling) — filtering has already been performed.
- The data are raw GCxGC-TOFMS instrument files that have not yet been peak-picked; run peak detection and annotation first before applying these QC thresholds.
- The dataset lacks QC samples or run-order information, so the Pearson |r|>0.5 and QC RSD 0.3–0.5/0.5 criteria cannot be evaluated.

## Inputs

- GCxGC-TOFMS detected-peak table from plasma cholecystectomy samples (691 peaks), with QC and run-order metadata
- Methods text and S2 Table describing exclusion criteria and thresholds
- Reported retained-metabolite counts and annotation class fractions (69/8/18/100 of 195)

## Outputs

- peak_filter_audit.csv — one row per exclusion/replacement criterion with threshold, action, and reported count/percentage
- Annotation-class breakdown table for the 195 retained metabolites (putative 35.4%, class-assigned 4.1%, uncertain 9.2%, unknown 51.3%)
- List of the 10 PCA outlier metabolites flagged for missing-value replacement

## How to apply

Parse the paper's 'Data Processing And Metabolite Identification' methods and S2 Table to reconstruct the ordered exclusion criteria: split peaks, low-quality peak/mass spectra, column bleed artifacts, internal-standard peaks, Pearson run-order correlation >|0.5|, QC RSD > 0.5 with manual review of the RSD 0.3–0.5 zone, and PCA (centered/UV-scaled) values > 4 flagged as outliers with replacement applied at > 5. Build a structured audit table (peak_filter_audit.csv) with one row per criterion, recording threshold, action (exclude vs. replace), and reported count. Record the annotation breakdown of the 195 retained metabolites: 69/195 (35.4%) putative annotation, 8/195 (4.1%) assigned metabolite class, 18/195 (9.2%) uncertain, 100/195 (51.3%) unknown. Validate arithmetically that 69 + 8 + 18 + 100 = 195 and 35.4 + 4.1 + 9.2 + 51.3 = 100.0, and confirm 10 metabolites were PCA outliers before the retained set fed the PCA/OPLS-DA modeling (e.g., separation of S. Typhi vs. S. Paratyphi A carriage, p = 0.0031).

## Related tools

- **GCxGC-TOFMS** (Instrument platform that generated the 691-peak plasma metabolite dataset to which the PCA outlier (value > 4, replacement > 5) and QC-RSD filtering criteria were applied)

## Examples

```
import pandas as pd; audit = pd.read_csv('peak_filter_audit.csv'); outliers = audit[(audit.pca_value > 4) & (audit.pca_value <= 5)]  # 10 PCA outlier metabolites flagged for replacement
```

## Evaluation signals

- Arithmetic invariant: 69 + 8 + 18 + 100 = 195 retained metabolites.
- Percentage invariant: 35.4 + 4.1 + 9.2 + 51.3 = 100.0%.
- Exactly 10 metabolites are recorded as PCA outliers in the audit table.
- Every exclusion row carries one of the paper's thresholds (split peak, low-quality spectrum, run-order |r| > 0.5, QC RSD > 0.5, PCA > 4/5); no unlisted criteria appear.
- Final retained count is 195 of 691 detected peaks (~28% retention), matching the reported filter outcome.

## Limitations

- The study's small sample size was a major limitation and restricted validation of downstream biomarker models built on the filtered 195 metabolites.
- No independent validation cohort was available, so filtered feature sets could not be externally confirmed.
- More than half of retained metabolites (100/195, 51.3%) had unknown identity, limiting biological interpretation of filter-surviving features.
- Thresholds (RSD 0.3–0.5 manual review, PCA > 4 vs. > 5) are study-specific conventions and may not transfer directly to other cohorts or platforms.

## Evidence

- [methods] Pearson run-order correlation >|0.5|, QC RSD > 0.5 with manual review of RSD 0.3–0.5, PCA centered/UV-scaled value > 4 with replacement at > 5: [evidence span withheld — anchor status: not_found]
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [supplementary] the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031): "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [discussion] The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
