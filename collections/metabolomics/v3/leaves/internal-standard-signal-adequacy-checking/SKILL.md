---
name: internal-standard-signal-adequacy-checking
description: Use when preprocessing targeted full-scan MSI-CE-MS runs where peak annotation
  depends on internal standards (or other reliable landmark peaks) with sufficient
  signal-to-noise; verify IS signal adequacy before trusting peak areas or reference
  ranges.
when_to_use_negative:
- When the input is already a clean peak-area or reference-range table — this check
  is upstream, on raw mzML/EIE signals.
- When the analyte of interest does not rely on internal standards or landmark peaks
  for migration-time prediction (e.g. purely de novo feature detection outside targeted
  MSI-CE-MS).
- When matrix effects or peak misannotation arise from co-migrating analytes (interference.comigration.threshold
  handling) rather than from weak IS signals.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: R software that performs the internal-standard adequacy and peak-annotation
    preprocessing of MSI-CE-MS mzML files
  repo: https://github.com/PBM-Group/PeakMeister
- name: MSI-CE-MS
  role: High-throughput separation platform whose runs are being quality-checked via
    IS signal adequacy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:2b5058553699732309ddf20c932a5368342045045df6c6d9089cfee2b8cea76a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# internal-standard-signal-adequacy-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A quality-control step in PeakMeister's MSI-CE-MS preprocessing that verifies internal-standard signals are adequate (e.g. adequate SNR, lock-mass counts, and migration-time prediction) before metabolite peak annotation; inadequate IS signals are the main cause of peak misannotation and outlier files.

## When to use

Use when preprocessing targeted full-scan MSI-CE-MS runs where peak annotation depends on internal standards (or other reliable landmark peaks) with sufficient signal-to-noise; verify IS signal adequacy before trusting peak areas or reference ranges.

## When NOT to use

- When the input is already a clean peak-area or reference-range table — this check is upstream, on raw mzML/EIE signals.
- When the analyte of interest does not rely on internal standards or landmark peaks for migration-time prediction (e.g. purely de novo feature detection outside targeted MSI-CE-MS).
- When matrix effects or peak misannotation arise from co-migrating analytes (interference.comigration.threshold handling) rather than from weak IS signals.

## Inputs

- mzML files of MSI-CE-MS runs (serum/QC samples) placed in the 'mzML Files' folder
- PeakMeister 'Mass List and Parameters.xlsx' with Sheet 2 (Reference Mass List: internal standards/landmark peaks, min.mt.min, max.mt.min) and Sheet 3 (ref.mass.counts, apply.mass.correction)
- Extracted ion electropherograms and peak-annotation/interference records produced by PeakMeister

## Outputs

- Per-run verdict that internal-standard/lock-mass signals are adequate or inadequate (candidate outlier file)
- Corrected/adjusted peak-annotation table and migration-index table once IS adequacy is confirmed
- List of runs/files to exclude or re-run (e.g. the ornithine-15N outlier file)

## How to apply

In PeakMeister, confirm each internal standard (or reliable landmark peak listed as 'Reference' in the Reference Mass List) has a strong, consistently detected peak whose area is always the most intense within its extracted ion electropherogram. Check lock-mass peak height against the ref.mass.counts minimum so mass correction is actually applied, and verify that migration-time prediction (which uses a panel of 11 internal standards via migration time indices, with a migration.window.seconds search window around the expected MT) yields peaks that fall inside the expected ± migration window rather than being missed or misannotated. Flag runs where a stable-isotope IS signal drops low enough to trigger misannotation — the article reports one outlier ENANI-2019 file where 73 peaks were misannotated because of a low ornithine-15N internal standard signal. Re-run or exclude the file once the IS signal is judged inadequate; downstream outputs such as peak areas, '<LOD' calls (snr.threshold), and reference-range tables depend on this check.

## Related tools

- **PeakMeister** (R software that performs the internal-standard adequacy and peak-annotation preprocessing of MSI-CE-MS mzML files) — https://github.com/PBM-Group/PeakMeister
- **MSI-CE-MS** (High-throughput separation platform whose runs are being quality-checked via IS signal adequacy)

## Examples

```
Rscript code  # PeakMeister R script inside the project, after filling 'Mass List and Parameters.xlsx' and populating 'mzML Files'
```

## Evaluation signals

- Lock-mass peak height exceeds the ref.mass.counts minimum so apply.mass.correction is actually engaged.
- Each Reference-class IS peak is the most intense signal in its extracted ion electropherogram and survives minimum.peak.width.seconds filtering.
- Migration-time prediction places analyte peaks within the migration.window.seconds window, not flagged as '<LOD' or outside the snr.threshold.
- Zero or few peaks reannotated by the peak-space reanalysis step (peak.space.tolerance.percent) — sustained high misannotation counts (e.g. 73 peaks) indicate an inadequate IS signal.
- Downstream annotation accuracy reproduces the article's median accuracy >99.9%.

## Limitations

- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation.
- Low-abundance metabolites with signals near the method detection limit are prone to misannotation even when IS signals are otherwise adequate.
- PeakMeister currently has no user-friendly interface or real-time output, so adequacy review is retrospective (weeks/months after acquisition).
- Not yet validated for matrices other than serum (urine, saliva, stool).

## Evidence

- [other] misannotated owing to a low ornithine-15N internal standard signal: [evidence span withheld — anchor status: not_found]
