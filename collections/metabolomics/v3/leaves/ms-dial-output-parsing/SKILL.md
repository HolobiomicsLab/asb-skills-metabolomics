---
name: ms-dial-output-parsing
description: Use when you have run Nextflow4MS-DIAL (or pulled its results folder)
  and need to quantify the MS-DIAL peak table—e.g., total peaks, true-positive counts,
  or percentages—for the 10-sample MTBLS733 dataset.
when_to_use_negative:
- You want to parse the MS-FLO output table (a separate, downstream peak table produced
  after duplicate-peak, adduct/isotope flagging, alignment, and contaminant-ion removal)
  rather than the raw MS-DIAL table.
- Your data has not been processed by the Nextflow4MS-DIAL workflow — the .msdial-to-.tsv
  renaming and the reported counts assume that specific pipeline's output format.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0153
tools:
- name: MS-DIAL
  role: Produces the .msdial peak table that this skill parses for peak counts and
    true-positive assessment.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow4MS-DIAL
  role: Runs the containerized workflow that generates the .msdial files under the
    results/ folder, which this skill then reads.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: pandas
  role: Loads the renamed .tsv peak table and computes peak counts and true-positive
    percentages.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:059548c6e3886ffc02bd7f12b50f4ee7eaaa777e6c927f9be0db6e3550d7cbae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow4MS-DIAL
  - pandas
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ms-dial-output-parsing

## Summary

Load and interpret the peak tables produced by the MS-DIAL component of the Nextflow4MS-DIAL workflow, including the renamed .tsv export files, to count total detected features and true positives on the MetaboLights MTBLS733 test data.

## When to use

Use when you have run Nextflow4MS-DIAL (or pulled its results folder) and need to quantify the MS-DIAL peak table—e.g., total peaks, true-positive counts, or percentages—for the 10-sample MTBLS733 dataset.

## When NOT to use

- You want to parse the MS-FLO output table (a separate, downstream peak table produced after duplicate-peak, adduct/isotope flagging, alignment, and contaminant-ion removal) rather than the raw MS-DIAL table.
- Your data has not been processed by the Nextflow4MS-DIAL workflow — the .msdial-to-.tsv renaming and the reported counts assume that specific pipeline's output format.

## Inputs

- MS-DIAL output peak table (.msdial file renamed to .tsv) from the Nextflow4MS-DIAL results folder
- Reference feature set from Li et al. (2018) for true-positive matching

## Outputs

- Total peak count for MS-DIAL (e.g., 25,447)
- True-positive peak count and percentage (e.g., 1,031 / 4.05%)
- peak_counts_summary.csv with the four counts and two percentages

## How to apply

The MS-DIAL process inside Nextflow4MS-DIAL emits .msdial peak-table files; in the repository's results folder these have been renamed to .tsv so they can be opened with spreadsheet software or read into pandas. Load the renamed .tsv with pandas and count the total number of rows (peaks) — for the MTBLS733 test run this yields 25,447 peaks. To compute true positives, match the peak m/z and retention-time pairs against the reference feature set from Li et al. (2018) and express the count as a percentage of total peaks — MS-DIAL gave 1,031 true positives, i.e. 4.05%. Validate by comparing your computed counts against the article-reported values, and note that repeated executions of the workflow produced identical results, so any deviation from 25,447 / 1,031 signals a configuration or parameter drift rather than stochastic variation.

## Related tools

- **MS-DIAL** (Produces the .msdial peak table that this skill parses for peak counts and true-positive assessment.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow4MS-DIAL** (Runs the containerized workflow that generates the .msdial files under the results/ folder, which this skill then reads.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **pandas** (Loads the renamed .tsv peak table and computes peak counts and true-positive percentages.)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('results/msdial_output.tsv', sep='\t'); print(len(df))"
```

## Evaluation signals

- Computed total MS-DIAL peak count must equal 25,447 for the MTBLS733 dataset.
- Computed true-positive count must equal 1,031, which is 4.05% of the total.
- Values written to peak_counts_summary.csv must match the article-reported numbers exactly.
- Re-running the workflow five times should yield identical peak counts, confirming reproducibility.

## Limitations

- The test data included only 10 samples, limiting generalizability of the counts.
- The results are specific to MS-DIAL version 4.48 bundled in the workflow; other versions may produce different peak tables.
- Only .mzML and .abf input formats are supported; other raw formats must be converted first.

## Evidence

- [results] true positive percentage: [evidence span withheld — anchor status: not_found]
