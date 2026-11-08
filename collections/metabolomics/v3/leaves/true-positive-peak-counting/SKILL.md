---
name: true-positive-peak-counting
description: Use when validating or benchmarking peak-detection output (e.g., MS-DIAL
  or MS-FLO peak tables) against a reference feature set by reporting total peaks,
  true-positive peaks, and the true-positive fraction.
when_to_use_negative:
- No reference feature set / ground truth exists for the dataset, so true positives
  cannot be determined and only raw peak counts would be available
- The goal is downstream statistical analysis (e.g., group comparison or biomarker
  selection) rather than validating peak-detection counts
- Peak tables come from tools other than MS-DIAL/MS-FLO output of the Nextflow4MS-DIAL
  workflow, since column schemas and flagging conventions differ
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: Nextflow4MS-DIAL
  role: Containerized Nextflow workflow (MS-DIAL 4.48 + MS-FLO) whose 'results' folder
    supplies the peak tables being counted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Produces the primary peak table whose total (25,447) and true-positive (1,031)
    peaks are counted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: Produces the filtered/aligned peak table (after duplicate flagging, adduct/isotope
    flagging, alignment, contaminant ion removal) whose total (25,414) and true-positive
    (1,030) peaks are counted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine used to execute MS-DIAL and MS-FLO reproducibly (Docker profile)
    before counting
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:9bb43bec1810f6a1e15dc93d9ef19d80ae4788f632c55260e428047e09e2f7f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# true-positive-peak-counting

## Summary

A validation procedure for LC-MS metabolomics peak-detection output that counts total detected peaks and true-positive peaks against a reference feature set, expressing true positives as a percentage of each tool's total count. Grounded in the Nextflow4MS-DIAL paper's benchmark of MS-DIAL vs MS-FLO on the MTBLS733 dataset.

## When to use

Use when validating or benchmarking peak-detection output (e.g., MS-DIAL or MS-FLO peak tables) against a reference feature set by reporting total peaks, true-positive peaks, and the true-positive fraction.

## When NOT to use

- No reference feature set / ground truth exists for the dataset, so true positives cannot be determined and only raw peak counts would be available
- The goal is downstream statistical analysis (e.g., group comparison or biomarker selection) rather than validating peak-detection counts
- Peak tables come from tools other than MS-DIAL/MS-FLO output of the Nextflow4MS-DIAL workflow, since column schemas and flagging conventions differ

## Inputs

- MS-DIAL output peak table (.msdial file renamed to .tsv) from the Nextflow4MS-DIAL 'results' folder
- MS-FLO output peak table (post duplicate-peak flagging, adduct/isotope flagging, alignment, contaminant ion removal)
- Reference feature set from Li et al. 2018 (Analytica Chimica Acta 1029, 50-57) for the MTBLS733 dataset
- 10-sample LC-HRMS MTBLS733 dataset (five samples in each of two groups)

## Outputs

- Total peak count per tool (MS-DIAL: 25,447; MS-FLO: 25,414)
- True-positive peak count per tool (MS-DIAL: 1,031; MS-FLO: 1,030)
- True-positive fraction per tool as a percentage (both 4.05%)
- peak_counts_summary.csv containing the four counts and two percentages

## How to apply

Run the containerized Nextflow4MS-DIAL workflow (Nextflow with the Docker profile, bundling MS-DIAL version 4.48 and MS-FLO) on the 10-sample MTBLS733 dataset from Li et al., or retrieve the pre-computed peak tables from the repository's 'results' folder. Load the MS-DIAL output peak table — whose .msdial files have been renamed to .tsv — with pandas and count total detected peaks. Load the MS-FLO output peak table (produced after duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal) and count its total peaks. Determine true-positive peaks in each table by matching against the reference feature set from the Li et al. 2018 evaluation study, and express each count as a percentage of that tool's total peaks. Write the four counts and two percentages to a summary CSV (e.g., peak_counts_summary.csv) for comparison against article-reported values.

## Related tools

- **Nextflow4MS-DIAL** (Containerized Nextflow workflow (MS-DIAL 4.48 + MS-FLO) whose 'results' folder supplies the peak tables being counted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Produces the primary peak table whose total (25,447) and true-positive (1,031) peaks are counted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Produces the filtered/aligned peak table (after duplicate flagging, adduct/isotope flagging, alignment, contaminant ion removal) whose total (25,414) and true-positive (1,030) peaks are counted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine used to execute MS-DIAL and MS-FLO reproducibly (Docker profile) before counting) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('results/msdial_peak_table.tsv', sep='\t'); print('total peaks:', len(df))"
```

## Evaluation signals

- MS-DIAL totals must reproduce the article values exactly: 25,447 total peaks, 1,031 true positives (4.05%)
- MS-FLO totals must reproduce the article values exactly: 25,414 total peaks, 1,030 true positives (4.05%)
- Both tools should yield the same true-positive fraction (4.05%), so a mismatched percentage flags a mis-matched reference set
- Repeated executions should give identical counts — the paper reports five runs each producing identical results
- peak_counts_summary.csv should contain exactly four counts and two percentages matching the reported values

## Limitations

- The benchmark uses only 10 samples, a small test set
- Few LC-MS datasets have been processed with both MS-DIAL and MS-FLO under transparent parameter settings, limiting convenient reproduction of steps
- No quantitative measure of scalability or reproducibility was reported by the authors
- The reference feature set comes from a specific prior study (Li et al. 2018); true-positive fractions are only meaningful for datasets with such ground truth

## Evidence
