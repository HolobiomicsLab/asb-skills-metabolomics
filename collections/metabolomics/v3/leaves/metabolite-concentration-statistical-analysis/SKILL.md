---
name: metabolite-concentration-statistical-analysis
description: Use when you have a metabolite-by-sample relative concentration matrix
  (e.g. 40 MDS family plasma samples x 38 metabolites from NMRQNet) and need to identify
  metabolites that differ significantly between patient and control groups.
when_to_use_negative:
- When the task is metabolite identification/quantification from raw NMR spectra rather
  than downstream statistics — use the NMRQNet EDTA plasma analysis pipeline (tutorial_for_EDTA_plasma.ipynb)
  first to obtain the concentration matrix.
- When metabolite predictions include suspected false positives — the article requires
  a final manual visualization check to filter false-positive metabolites before trusting
  downstream statistical comparisons.
- For biofluids other than plasma (no biofluid-specific analysis pipeline had been
  developed; clinical-diagnosis use was explicitly flagged as future work).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: NMRQNet
  role: Produces the relative concentration matrix (csv) of dominant metabolites from
    1H NMR plasma spectra that this statistical workflow then analyzes; also provides
    the EDTA plasma tutorial notebook
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:05a34e144be75fff961cd53970b35a53d0ed32abbad9432b91bb383c6810a281
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-concentration-statistical-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical downstream analysis of NMRQNet-estimated relative metabolite concentrations from plasma 1H NMR spectra: PCA outlier screening, supervised OPLS-DA group separation, and per-metabolite two-sample t-tests to nominate biomarkers.

## When to use

Use when you have a metabolite-by-sample relative concentration matrix (e.g. 40 MDS family plasma samples x 38 metabolites from NMRQNet) and need to identify metabolites that differ significantly between patient and control groups.

## When NOT to use

- When the task is metabolite identification/quantification from raw NMR spectra rather than downstream statistics — use the NMRQNet EDTA plasma analysis pipeline (tutorial_for_EDTA_plasma.ipynb) first to obtain the concentration matrix.
- When metabolite predictions include suspected false positives — the article requires a final manual visualization check to filter false-positive metabolites before trusting downstream statistical comparisons.
- For biofluids other than plasma (no biofluid-specific analysis pipeline had been developed; clinical-diagnosis use was explicitly flagged as future work).

## Inputs

- NMRQNet-estimated relative concentration matrix (csv) from the EDTA plasma analysis pipeline, e.g. 40 MDS family plasma samples x 38 metabolites
- Patient vs. control group labels (e.g. MDS patient vs. healthy family member)

## Outputs

- Per-metabolite t-statistics and p-values table (e.g. mds_metabolite_ttest_results.csv)
- PCA outlier screening result (which samples are outliers prior to group comparison)
- OPLS-DA group-separation assessment (patient vs. control)
- Biomarker nomination with direction of change and p-value (e.g. acetate elevated in MDS patients, p=0.016)
- Boxplot of the nominated metabolite's concentrations (Fig. 6c-style)

## How to apply

Load the NMRQNet-estimated relative concentration matrix from the csv output of the EDTA plasma analysis pipeline (e.g. 40 Mecp2 duplication syndrome family plasma samples x 38 metabolites). First apply PCA on the concentration matrix to screen for outlier samples prior to any group comparison, since outlier spectra can drive apparent separation. Then fit an OPLS-DA model, a supervised projection using the patient vs. control labels, to assess group separation across the metabolite concentrations. Finally run two-sample t-tests on each metabolite's relative concentration between the two groups and rank metabolites by p-value. In the MDS case study this workflow nominated acetate as the significant biomarker, significantly elevated in MDS patients with t-test p=0.016, and results were written to mds_metabolite_ttest_results.csv with a Fig. 6c-style boxplot of acetate concentrations. Interpret nominated biomarkers as candidate only: the article notes further experiments were needed to confirm them.

## Related tools

- **NMRQNet** (Produces the relative concentration matrix (csv) of dominant metabolites from 1H NMR plasma spectra that this statistical workflow then analyzes; also provides the EDTA plasma tutorial notebook) — github.com/LiuzLab/NMRQNet

## Evaluation signals

- t-test p-value for the nominated biomarker matches the expected value (acetate p=0.016) on the 40-sample MDS family dataset
- PCA screening is performed before group comparisons and no identified outlier samples remain in the patient/control statistics
- OPLS-DA shows separation between patient and control labels consistent with the significant t-test finding
- Output csv (mds_metabolite_ttest_results.csv) contains t-statistics and p-values for all 38 metabolites
- Direction of change is consistent: acetate concentrations are significantly elevated in the MDS patient group, and a Fig. 6c-style boxplot shows this shift

## Limitations

- Nominated biomarkers are candidate only — further experiments were needed to confirm the potential biomarkers.
- NMRQNet regularizes insufficiently on false-positive predictions; a final manual visualization check is required to filter metabolites with false-positive predictions before downstream statistics.
- The downstream statistical analysis was demonstrated on the MDS family plasma case; no automated pre/post filtering for false positives exists yet.

## Evidence
