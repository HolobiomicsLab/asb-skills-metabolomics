---
name: within-condition-correlation-threshold-screening
description: Use when quality-controlling an expression dataset where samples should
  cohere within biological conditions — e.g. before differential-expression analysis,
  to verify same-organ, same-tumor-status samples show pairwise Pearson correlations
  >0.70 and to flag low-expression outlier samples.
when_to_use_negative:
- Input is not sample-level expression data with condition metadata — this skill groups
  by organ/tumor status and is meaningless without it
- Goal is gene-level rather than sample-level assessment — e.g. Spearman correlation
  of a single gene (such as CDKN2A) against all genes is a different analysis
- The dataset has already been filtered and all within-condition pairwise Pearson
  correlations are known to exceed 0.70
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOG (MetaOmGraph)
  role: Computes pairwise Pearson correlations among samples from the same biological
    condition and supports metadata-driven sample filtering for outlier removal
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional back-end for additional statistical analyses of data subsets sent
    from MOG
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:f650af217e75ece47f7df557162941872a67ef6e0c7034751c250ba41f45be60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MOG (MetaOmGraph)
  - MetaOmGraph
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# within-condition-correlation-threshold-screening

## Summary

A sample-level QC skill that uses MOG to compute pairwise Pearson correlations among samples from the same biological condition (organ and tumor status) and screens them against a >0.70 acceptance threshold to detect aberrant samples such as the LUAD outlier removed from the Hu-cancer-RNASeq-dataset.

## When to use

Use when quality-controlling an expression dataset where samples should cohere within biological conditions — e.g. before differential-expression analysis, to verify same-organ, same-tumor-status samples show pairwise Pearson correlations >0.70 and to flag low-expression outlier samples.

## When NOT to use

- Input is not sample-level expression data with condition metadata — this skill groups by organ/tumor status and is meaningless without it
- Goal is gene-level rather than sample-level assessment — e.g. Spearman correlation of a single gene (such as CDKN2A) against all genes is a different analysis
- The dataset has already been filtered and all within-condition pairwise Pearson correlations are known to exceed 0.70

## Inputs

- Hu-cancer RNA-Seq MOG project (expression matrix of 18,212 genes with associated samples)
- Sample metadata describing biological condition (organ/cancer type and tumor/non-tumor status)

## Outputs

- Full sample-sample Pearson correlation matrix saved as correlation_matrix.csv
- Summary of the flagged LUAD outlier sample saved as luad_outlier_report.csv
- Filtered dataset with the LUAD outlier sample removed

## How to apply

Load the Hu-cancer RNA-Seq MOG project together with its sample metadata, then group samples by biological condition (cancer type/organ and tumor/non-tumor status). Use MOG to compute pairwise Pearson correlations among samples from the same biological condition. Screen all within-condition correlation values against the acceptance threshold of >0.70; any sample whose correlations with condition-mates fall well below 0.70 should additionally be checked for very low expression values before being flagged. In the published case this identified exactly one lung adenocarcinoma (LUAD) sample, which the authors removed from the dataset. Save the full sample-sample Pearson correlation matrix as correlation_matrix.csv and a summary of the flagged LUAD outlier sample as luad_outlier_report.csv.

## Related tools

- **MOG (MetaOmGraph)** (Computes pairwise Pearson correlations among samples from the same biological condition and supports metadata-driven sample filtering for outlier removal) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional back-end for additional statistical analyses of data subsets sent from MOG)

## Evaluation signals

- All remaining within-condition pairwise Pearson correlations exceed 0.70
- Exactly one LUAD sample is flagged as the outlier and its flag is justified by abnormally low expression values
- The saved correlation_matrix.csv contains pairwise Pearson values for every pair of samples in the same condition
- luad_outlier_report.csv exists and identifies the removed sample along with its low-expression evidence
- Dataset after filtering no longer contains the flagged LUAD sample

## Limitations

- The >0.70 threshold was calibrated on the Hu-cancer-RNASeq-dataset; it is a reported acceptance criterion, not a universal cutoff
- MOG is a Java GUI workbench (requires Java Runtime Environment 8 or higher; R 3.4+ optional) — automation of this screening requires interactive exploration or scripting outside MOG
- A sub-threshold correlation alone is not proof of poor sample quality; the article corroborated the LUAD outlier by checking for very low expression values

## Evidence

- [results] Correlation computation with MOG: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
- [results] Threshold finding and LUAD removal: "except one sample from lung adenocarcinoma (LUAD), which we removed from the dataset"
- [results] Outlier correlation evidence: [evidence span withheld — anchor status: not_found]
- [readme] MOG availability and purpose: "MetaOmGraph (MOG) is a Java software to interactively explore and visualize large datasets."
