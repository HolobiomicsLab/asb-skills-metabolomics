---
name: alignment-loss-quantification
description: Use when you have both unaligned and aligned NPP output tables (e.g.
  XCMS_unaligned_run3.csv and XCMS_aligned_run3.csv) plus a generated benchmark, and
  you need to quantify peaks lost or misaligned during sample alignment.
when_to_use_negative:
- No benchmark dataset has been generated yet — mzRAPP's assessment requires a validated
  benchmark; raw mzML files alone are insufficient.
- The output to evaluate is not an NPP peak/alignment table from a supported tool
  (XCMS, MZmine 2, MS-DIAL, etc.) but already a manually validated or targeted feature
  table.
- Only a single (aligned) NPP output is available — alignment loss cannot be quantified
  without the unaligned counterpart.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: Runs the NPP assessment that matches the benchmark against aligned/unaligned
    output tables and reports alignment error counts
  repo: https://github.com/YasinEl/mzRAPP
- name: XCMS
  role: Non-targeted pre-processing tool whose aligned and unaligned peak tables are
    being evaluated for alignment losses
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:9184b2b4f5399a8bb53f5e725700a7264d09e3be20a683e1fe5a4d4080fc3eb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# alignment-loss-quantification

## Summary

Quantifies how many benchmark peaks are lost or misaligned when non-targeted pre-processing (NPP) outputs are aligned across samples, using mzRAPP to match an XCMS (or similar tool) aligned and unaligned output against a validated benchmark.

## When to use

Use when you have both unaligned and aligned NPP output tables (e.g. XCMS_unaligned_run3.csv and XCMS_aligned_run3.csv) plus a generated benchmark, and you need to quantify peaks lost or misaligned during sample alignment.

## When NOT to use

- No benchmark dataset has been generated yet — mzRAPP's assessment requires a validated benchmark; raw mzML files alone are insufficient.
- The output to evaluate is not an NPP peak/alignment table from a supported tool (XCMS, MZmine 2, MS-DIAL, etc.) but already a manually validated or targeted feature table.
- Only a single (aligned) NPP output is available — alignment loss cannot be quantified without the unaligned counterpart.

## Inputs

- Unaligned NPP peak table (e.g. XCMS_unaligned_run3.csv)
- Aligned NPP peak table (e.g. XCMS_aligned_run3.csv)
- Benchmark dataset CSV (Benchmark.csv) generated from centroided mzML files with user.rtmin/user.rtmax boundaries
- Sample-group CSV with sample_name and sample_group columns (used upstream in benchmark generation)

## Outputs

- NPP assessment metrics split into Post Alignment, Peak Picking, and Alignment overview boxes
- Alignment error counts (proportion of peaks lost during alignment)
- Exported metrics CSV (e.g. xcms_run3_assessment_metrics.csv)

## How to apply

Launch the mzRAPP shiny interface with library(mzRAPP); callmzRAPP(). In the 'Setup NPP assessment' tab choose the non-targeted tool (e.g. XCMS) and supply the unaligned file, the aligned file, and the benchmark file (Benchmark.csv exported from the benchmark-generation workflow, e.g. built from MTBLS267 _POS.mzML files). mzRAPP matches benchmark peaks to the NPP outputs via the compare_peaks/match_peaks_to_benchmark pipeline, which links each benchmark peak to its counterpart in the unaligned peak list and then checks whether it appears in the aligned feature table. Inspect the three overview boxes in the 'View NPP assessment' tab — Post Alignment, Peak Picking, and Alignment — and record the reported metric ranges. Export the computed metrics (e.g. to xcms_run3_assessment_metrics.csv). Because mzRAPP bootstraps confidence intervals, slight run-to-run variation in the metrics is expected and acceptable.

## Related tools

- **mzRAPP** (Runs the NPP assessment that matches the benchmark against aligned/unaligned output tables and reports alignment error counts) — https://github.com/YasinEl/mzRAPP
- **XCMS** (Non-targeted pre-processing tool whose aligned and unaligned peak tables are being evaluated for alignment losses)

## Examples

```
library(mzRAPP); callmzRAPP()  # open the GUI, then in 'Setup NPP assessment' supply XCMS_unaligned_run3.csv, XCMS_aligned_run3.csv and Benchmark.csv
```

## Evaluation signals

- Post-alignment detection falls in the reported ~93–99% range of benchmark peaks found after alignment
- Degenerated isotopologue ratios (peak abundance quality) fall in the reported 3–20% range
- Peaks lost during alignment fall in the reported ~4–10% range
- The exported metrics file (e.g. xcms_run3_assessment_metrics.csv) is produced and populated with all three overview-box metrics
- Re-running the assessment yields slightly different but overlapping values, consistent with mzRAPP's bootstrapped confidence intervals

## Limitations

- mzRAPP bootstraps confidence intervals, so repeated assessments show an element of randomness; metrics outside but near the reported ranges may still be acceptable.
- Alignment loss can only be assessed when both unaligned and aligned NPP outputs from the same run are supplied.
- The benchmark itself is filtered: only isotopologues with the most-abundant plus at least one additional isotopologue detected, area/height within 30% of the predicted value, and Pearson correlation ≥ 0.85 against the highest isotopologue are retained, so rare or poorly shaped peaks are not represented in alignment-loss counts.
- mzRAPP supports five of the most commonly used pre-processing tools; outputs from unsupported tools must be reformatted before assessment.

## Evidence

- [readme] The resulting benchmark dataset is used to extract different performance metrics for NPP performed on the same mzML files.: [evidence span withheld — anchor status: external_doc]
- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [intro] Compare NPP outputs from different tools via matching against a benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Download the XCMS- and MZmine 2-output files from ucloud: "Download the XCMS- and MZmine 2-output files from [ucloud](https://ucloud.univie.ac.at/index.php/s/BjF24f7q4jeFJH3)"
- [readme] mzRAPP can utilize user-provided information on a set of molecules (the more molecules the better) with known retention behavior: "mzRAPP can utilize user-provided information on a set of
molecules (the more molecules the better) with known retention behavior."
