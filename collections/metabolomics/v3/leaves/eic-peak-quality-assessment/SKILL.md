---
name: eic-peak-quality-assessment
description: Use when a feature-extraction step (e.g. DaDIA.R rescued features or
  significant metabolic features from DDA/DaDIA LC-MS data) yields candidate peaks
  whose automated detection quality needs validation before downstream annotation
  or statistical comparison.
when_to_use_negative:
- When the goal is purely automated, high-throughput feature quantification with no
  manual validation budget — EIC inspection is a targeted QC step, not a substitute
  for algorithmic peak picking.
- When features already carry validated MS2 spectra and have been annotated with high
  dot-product scores (> 0.7) against a spectral library — those features have an independent
  quality signal and manual EIC review adds little.
- When the input is already a curated, manually reviewed feature table (i.e. no unvalidated
  rescued features remain).
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DaDIA.R
  role: Generates the feature table (including rescued features from combining DDA
    and DIA data) whose EICs are then manually inspected for peak quality.
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Upstream centWave-based feature extraction on the mzXML files that produces
    the candidate peaks and their retention-time windows used to render EICs.
- name: R
  role: Environment in which the raw chromatograms and EIC plots are rendered and
    the quality labels are tabulated.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:a704dd63d841d77c2578ba94caebb31b093ec03ea43d4116da1919e42aa34633
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# eic-peak-quality-assessment

## Summary

Manually inspect extracted-ion chromatograms (EICs) of rescued or significant metabolic features to verify that chromatographic peaks are genuine rather than noise or mis-extracted artifacts, classifying each EIC as true-positive, false-positive, or undetermined.

## When to use

Use when a feature-extraction step (e.g. DaDIA.R rescued features or significant metabolic features from DDA/DaDIA LC-MS data) yields candidate peaks whose automated detection quality needs validation before downstream annotation or statistical comparison.

## When NOT to use

- When the goal is purely automated, high-throughput feature quantification with no manual validation budget — EIC inspection is a targeted QC step, not a substitute for algorithmic peak picking.
- When features already carry validated MS2 spectra and have been annotated with high dot-product scores (> 0.7) against a spectral library — those features have an independent quality signal and manual EIC review adds little.
- When the input is already a curated, manually reviewed feature table (i.e. no unvalidated rescued features remain).

## Inputs

- Rescued or significant metabolic feature table (m/z, retention time, sample intensities) from XCMS/DaDIA.R
- Raw chromatographic data in mzXML or vendor format for each analyzed sample
- Method-blank and pooled-QC sample intensities for background context

## Outputs

- Per-feature EIC quality label: true-positive, false-positive, or undetermined
- Aggregate quality rates (e.g. true-positive rate, false-positive rate) for a feature set
- Annotated EIC images or plots supporting each label

## How to apply

After feature extraction and significance filtering (e.g. FC > 1.5 or < 0.67 with paired two-tailed t-test p < 0.05), pull the raw chromatographic data for each candidate feature and render an EIC at the feature's m/z and retention-time window using the original vendor or converted mzXML files. Inspect each EIC visually and judge peak shape: peaks with clean, symmetric, reproducible shapes across biological samples are scored true-positive; EICs showing only flat baseline, spike artifacts, or peaks absent from expected samples are scored false-positive; ambiguous cases are scored undetermined. In the DaDIA urine study this manual check of rescued features yielded a 71% true-positive rate and a 17% false-positive rate (12% undetermined), while for the 486 DaDIA-unique significant features in the leukemia plasma study it yielded 88% true-positive and 5% false-positive. Use the resulting per-feature quality labels to recompute or bound the reliability of automated feature counts, and retain the labeled EIC table alongside the feature table so that downstream annotation (e.g. dot-product MS2 matching against MoNA with score > 0.7) inherits a quality flag.

## Related tools

- **DaDIA.R** (Generates the feature table (including rescued features from combining DDA and DIA data) whose EICs are then manually inspected for peak quality.) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Upstream centWave-based feature extraction on the mzXML files that produces the candidate peaks and their retention-time windows used to render EICs.)
- **R** (Environment in which the raw chromatograms and EIC plots are rendered and the quality labels are tabulated.)

## Examples

```
Rscript eic_quality_check.R --features significant_features.csv --raw mzxml/ --blank blanks.csv --out eic_labels.csv
```

## Evaluation signals

- Every significant feature in the comparison set carries an explicit true-positive / false-positive / undetermined label (no missing or 'auto' values).
- For the leukemia plasma study the aggregate quality rates converge to the reported values: 486 DaDIA-unique significant features with 88% true-positive and 5% false-positive EIC quality.
- For the urine rescued-feature set the aggregate quality rates converge to 71% true-positive, 17% false-positive, and 12% undetermined.
- The labeled EIC table row count equals the number of candidate features submitted for inspection (no silently dropped rows).
- Downstream annotation output (e.g. dot-product-scored metabolites) inherits or references the EIC quality label for each annotated feature.

## Limitations

- Manual EIC inspection is subjective — the article explicitly reports a residual 'undetermined' class (12% in the urine rescued set) where peak quality cannot be judged from the chromatogram alone.
- False-positive rescued features (17% in the urine study) indicate that automated rescue from DDA data produces EICs lacking good chromatographic peak shapes, so the label should not be assumed correct without inspection.
- The assessment depends on the underlying chromatographic separation quality; poor LC gradients or overloaded columns can make genuine peaks look ambiguous.

## Evidence

- [results] Manual EIC inspection of rescued features classifies peak quality into true-positive / false-positive / undetermined categories.: [evidence span withheld — anchor status: not_found]
- [results] In the leukemia plasma study, 486 DaDIA-unique significant features were manually validated via EIC quality assessment, yielding 88% true-positive and 5% false-positive rates.: [evidence span withheld — anchor status: not_found]
