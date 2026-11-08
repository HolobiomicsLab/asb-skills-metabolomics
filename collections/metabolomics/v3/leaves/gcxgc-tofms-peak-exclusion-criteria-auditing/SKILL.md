---
name: gcxgc-tofms-peak-exclusion-criteria-auditing
description: Use when auditing which GCxGC-TOFMS peaks were excluded from a metabolomics
  dataset and why — e.g. reconciling an initial peak count (691) against the final
  metabolite count (195) with per-criterion thresholds.
when_to_use_negative:
- Input is already a filtered feature table where exclusions were applied upstream
  — this skill audits the filter step itself, not downstream statistics.
- The analysis is about which metabolites discriminate carriers (OPLS-DA/ROC), not
  about why peaks were removed — use a modeling skill instead.
- No peak-level exclusion thresholds are reported or recoverable from the source text
  — an audit table without thresholds is unverifiable.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCxGC-TOFMS
  role: Instrument whose raw peak list (691 peaks) is the input the audit filters
    are applied to
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:5a7af0f545d7dfeffe74aaddc039ee544106f1d7e4ea2466b8907c637bf9692a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gcxgc-tofms-peak-exclusion-criteria-auditing

## Summary

Audits the GCxGC-TOFMS peak-exclusion and missing-value-replacement pipeline that reduced 691 detected plasma peaks to 195 retained metabolites, recording each filter criterion, threshold, and action, plus the annotation-class breakdown of the retained set.

## When to use

Use when auditing which GCxGC-TOFMS peaks were excluded from a metabolomics dataset and why — e.g. reconciling an initial peak count (691) against the final metabolite count (195) with per-criterion thresholds.

## When NOT to use

- Input is already a filtered feature table where exclusions were applied upstream — this skill audits the filter step itself, not downstream statistics.
- The analysis is about which metabolites discriminate carriers (OPLS-DA/ROC), not about why peaks were removed — use a modeling skill instead.
- No peak-level exclusion thresholds are reported or recoverable from the source text — an audit table without thresholds is unverifiable.

## Inputs

- Raw GCxGC-TOFMS peak list from plasma cholecystectomy samples (691 detected peaks)
- Methods text describing 'Data Processing And Metabolite Identification' exclusion criteria
- S2 Table peak/metadata reference

## Outputs

- peak_filter_audit.csv — one row per exclusion/retention criterion with columns: criterion, threshold, action (exclude/replace), reported count/percentage
- Annotation class breakdown rows: 69/195 putative, 8/195 class-assigned, 18/195 uncertain, 100/195 unknown
- Arithmetic validation flags for count vs. percentage consistency

## How to apply

Parse the methods text describing 'Data Processing And Metabolite Identification' to extract each exclusion criterion: split peaks, low-quality peaks or mass spectra, column bleed artifacts and internal standards, Pearson run-order correlation > |0.5|, QC RSD > 0.5 (with manual review for RSD 0.3–0.5), and PCA (centered/UV-scaled) values > 4 with replacement at > 5. Record for each criterion the threshold value and the action taken — exclusion or missing-value replacement (10 PCA outlier metabolites). Then tabulate the retained 195 metabolites by annotation class: 69 (35.4%) putative annotation, 8 (4.1%) assigned metabolite class, 18 (9.2%) uncertain identity, 100 (51.3%) unknown identity. Validate arithmetically that 69 + 8 + 18 + 100 = 195 and the percentages sum to 100.0%, saving the audit as a structured CSV.

## Related tools

- **GCxGC-TOFMS** (Instrument whose raw peak list (691 peaks) is the input the audit filters are applied to)

## Examples

```
python build_audit_table.py --counts 691 --retained 195 --classes 69,8,18,100 --output peak_filter_audit.csv
```

## Evaluation signals

- Retained metabolite count equals 195 and excluded count reconciles with 691 initial peaks
- Annotation classes sum to retained set: 69 + 8 + 18 + 100 = 195
- Percentage breakdown sums to 100.0% (35.4 + 4.1 + 9.2 + 51.3)
- Every exclusion row carries a numeric threshold (e.g. Pearson |r| > 0.5, QC RSD > 0.5, PCA > 4) or explicit manual-review note
- 10 PCA outlier metabolites are flagged for missing-value replacement rather than exclusion

## Limitations

- Small sample size limited the original study and by extension the stability of filter thresholds
- 51.3% of retained metabolites remained unknown identity, limiting biological interpretation of the audit
- Potential confounding from underlying gallbladder conditions could affect which peaks pass QC filters
- No independent validation cohort was available to verify filter robustness

## Evidence

- [other] Of the 195 retained metabolites, 69 (35.4%) had a putative annotation, 8 (4.1%) an assigned metabolite class, 18 (9.2%) uncertain identity, and 100 (51.3%) unknown identity: [evidence span withheld — anchor status: not_found]
- [other] exclusion was mainly due to split peaks, low-quality peaks or mass spectra, run-order correlation, and QC deviation: [evidence span withheld — anchor status: not_found]
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [discussion] The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
- [discussion] The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
