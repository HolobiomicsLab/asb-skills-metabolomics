---
name: count-consistency-verification-of-metabolite-cohorts
description: Use when a paper reports a peak-to-metabolite funnel (e.g. 691 peaks
  reduced to 195 metabolites) with an annotation-class breakdown (putative, assigned
  class, uncertain, unknown) and the counts/percentages must be audited before reuse.
when_to_use_negative:
- The paper reports only a final feature count with no per-criterion exclusion thresholds
  or annotation-class fractions to reconcile.
- The input is already a validated feature table (e.g. the 195-metabolite matrix)
  and the filtering audit has already been performed.
- The analysis goal is downstream statistical modeling (OPLS-DA, ROC) rather than
  auditing the filtering funnel itself.
edam_operation: http://edamontology.org/operation_3678
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GCxGC-TOFMS
  role: Instrument whose raw detected-peak counts (691) form the starting point of
    the filtering funnel being audited; the retained set is the 195 metabolites generated
    from GCxGC-TOFMS analysis of plasma samples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:a68b740b11552bedce1eb128c282292b5d246809723977de07c792c2c13dd6d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3678
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Count-consistency verification of metabolite cohorts

## Summary

Verifies that the arithmetic of a metabolite filtering/annotation cascade closes: exclusion criteria applied to 691 detected GCxGC-TOFMS peaks must reconcile with the 195 retained metabolites, and the annotation-class breakdown (69+8+18+100=195, percentages summing to 100.0%) must be internally consistent.

## When to use

Use when a paper reports a peak-to-metabolite funnel (e.g. 691 peaks reduced to 195 metabolites) with an annotation-class breakdown (putative, assigned class, uncertain, unknown) and the counts/percentages must be audited before reuse.

## When NOT to use

- The paper reports only a final feature count with no per-criterion exclusion thresholds or annotation-class fractions to reconcile.
- The input is already a validated feature table (e.g. the 195-metabolite matrix) and the filtering audit has already been performed.
- The analysis goal is downstream statistical modeling (OPLS-DA, ROC) rather than auditing the filtering funnel itself.

## Inputs

- Methods text describing 'Data Processing And Metabolite Identification' exclusion criteria and thresholds
- Reported peak and metabolite counts (691 detected peaks, 195 retained metabolites, 10 PCA outlier metabolites)
- Annotation-class breakdown (69, 8, 18, 100 of 195) with percentages (35.4%, 4.1%, 9.2%, 51.3%)
- Supplementary tables (e.g. S2 Table) listing the retained metabolites

## Outputs

- peak_filter_audit.csv — one row per exclusion/processing criterion with threshold, action, and reported count/percentage
- Annotation-class breakdown rows with exact fractions (69/195, 8/195, 18/195, 100/195) and percentages
- Arithmetic validation flags (counts sum to 195; percentages sum to 100.0%)

## How to apply

Extract the exclusion criteria and thresholds from the Methods data-processing text plus supplementary tables: split peaks, low-quality peaks or mass spectra, column bleed artifacts, internal standard removal, Pearson run-order correlation |r|>0.5, QC RSD > 0.5 (with manual review of RSD 0.3–0.5), and PCA centered/UV-scaled values > 4 (replacement at > 5, 10 outlier metabolites). Build a structured audit table (CSV) with one row per criterion, recording the threshold, the action taken (exclusion vs missing-value replacement), and the reported count or percentage. Add rows for the annotation-class breakdown of the 195 retained metabolites: 69/195 (35.4%) putative annotation, 8/195 (4.1%) assigned metabolite class, 18/195 (9.2%) uncertain identity, 100/195 (51.3%) unknown identity. Validate arithmetically that 69+8+18+100 = 195 and that 35.4+4.1+9.2+51.3 = 100.0, flagging any mismatch. The rationale is that downstream modeling (OPLS-DA on the 195-metabolite matrix) is only trustworthy if the filtering funnel and class fractions are internally consistent.

## Related tools

- **GCxGC-TOFMS** (Instrument whose raw detected-peak counts (691) form the starting point of the filtering funnel being audited; the retained set is the 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples)

## Evaluation signals

- Class-count invariant: 69 + 8 + 18 + 100 must equal 195 (the number of retained metabolites).
- Percentage invariant: 35.4 + 4.1 + 9.2 + 51.3 must sum to ~100.0%.
- Every audit-table row must carry an explicit threshold (e.g. |r|>0.5 run-order correlation, QC RSD > 0.5, PCA value > 4 with replacement at > 5) and an action (exclude vs missing-value replacement).
- Funnel monotonicity: the retained count (195) must be strictly less than the starting peak count (691).
- The 10 PCA outlier metabolites reported as missing-value-replaced should be reconcilable with the PCA threshold rows in the audit table.

## Limitations

- The paper's breakdown percentages are rounded to one decimal, so the sum check only closes approximately (35.4+4.1+9.2+51.3 = 100.0 here, but rounding can mask off-by-one count errors).
- The audit verifies internal consistency of reported numbers, not correctness of the filtering decisions themselves; per-peak exclusion records are not available from the reported text.
- The study was small and metabolite profiles were descriptive; the 195-metabolite cohort may not generalize to other populations without re-validation.

## Evidence

- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [methods] exclusion criteria from Methods: [evidence span withheld — anchor status: not_found]
- [methods] exclusion was mainly due to split peaks, low-quality peaks or mass spectra, run-order correlation, and QC deviation: [evidence span withheld — anchor status: not_found]
- [supplementary] downstream modeling context: "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
