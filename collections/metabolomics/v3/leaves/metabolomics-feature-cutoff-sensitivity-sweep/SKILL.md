---
name: metabolomics-feature-cutoff-sensitivity-sweep
description: Use when an LC/MS metabolomics experiment produces a significant feature
  list chosen by a univariate cutoff and you need to check whether mummichog's predicted
  metabolite network is stable to that cutoff choice.
when_to_use_negative:
- Metabolite identities are already confirmed by tandem mass spectrometry or authentic
  standards, so cutoff sensitivity of putative predictions is not the concern
- The goal is to predict functional activity from a fresh feature table rather than
  to stress-test an existing prediction's stability
- Data come from a low-resolution instrument where m/z-to-metabolite matching is too
  ambiguous to make cutoff sweeps informative
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: Core engine rerun at each cutoff to produce pathway/network predictions from
    the significant and reference feature lists
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: metDataModel
  role: Data models for metabolomics used by mummichog; structure the feature table
    and model inputs across the sweep
- name: JMS
  role: Annotation library used to structure MS1 pseudo-spectra and empirical compounds
    for the features matched at each cutoff
  repo: https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:89c57434ee2045fdfee7029ea7885f37bcf095b1beb925a3fc9392f8fc55991a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JMS
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-feature-cutoff-sensitivity-sweep

## Summary

A robustness procedure for mummichog pathway/network predictions on untargeted LC/MS feature tables: sweep the feature-selection p-value cutoff used to build the significant feature list and quantify how the predicted metabolite sets overlap across cutoffs.

## When to use

Use when an LC/MS metabolomics experiment produces a significant feature list chosen by a univariate cutoff and you need to check whether mummichog's predicted metabolite network is stable to that cutoff choice.

## When NOT to use

- Metabolite identities are already confirmed by tandem mass spectrometry or authentic standards, so cutoff sensitivity of putative predictions is not the concern
- The goal is to predict functional activity from a fresh feature table rather than to stress-test an existing prediction's stability
- Data come from a low-resolution instrument where m/z-to-metabolite matching is too ambiguous to make cutoff sweeps informative

## Inputs

- LC/MS feature table (m/z, retention time, p-value from the statistical test) after quality control, e.g. the moDC table in Dataset S1 with 7,995 features
- Primary p-value cutoff used for feature selection and the range of cutoff values to sweep
- Human metabolic model built from KEGG, UCSD Recon1 and the Edinburgh human metabolic network (default mummichog model)
- Optional MS mode specification (positive or negative) for adduct enforcement

## Outputs

- Per-cutoff predicted activity networks (qualified metabolites with confidence ratings, isotopic derivatives and adducts)
- Per-cutoff pathway analysis and network module analysis results (JSON strings, result tables, figures)
- Summary table of pairwise overlap between predicted metabolite sets across all cutoffs

## How to apply

Split the LC/MS feature table into a significant list and the full reference list that mummichog requires, using the paper's primary cutoff (on the moDC data, 601 of 7,995 QC-passing features). Run the full mummichog pipeline at the primary cutoff: Fisher's exact test with EASE adjustment and a Gamma-modeled permutation null for pathway enrichment, plus Newman's spectral split on one-to-four-reaction subnetworks for module activity. Then sweep the cutoff over a range (as in Figure S7), rerunning the pathway and module analyses at each value, collecting predicted metabolites, their isotopic derivatives and adducts, and filtering by the confidence rating (e.g. co-occurrence of M+H[1+] and M(C13)+H[1+] forms confers high confidence). Compare predicted metabolite sets across cutoffs by pairwise overlap to assess stability, validating against the reported stable network of 60 metabolites (19 of which were not in the original annotation). Rationale: the article reports that predictions are largely consistent across the tested cutoffs, so the sweep establishes that downstream interpretation does not hinge on the exact threshold.

## Related tools

- **mummichog** (Core engine rerun at each cutoff to produce pathway/network predictions from the significant and reference feature lists) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **metDataModel** (Data models for metabolomics used by mummichog; structure the feature table and model inputs across the sweep)
- **JMS** (Annotation library used to structure MS1 pseudo-spectra and empirical compounds for the features matched at each cutoff) — https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks

## Examples

```
python3 -m mummichog.main -f mummichog/tests/testdata0710.txt -o t3
```

## Evaluation signals

- Per-cutoff predicted metabolite counts should bracket the reported value of 60 for the moDC data
- Pairwise overlaps between predicted metabolite sets at different cutoffs should be large, reproducing the Figure S7 stability claim
- Each prediction should carry a confidence rating derived from isotopic/adduct co-occurrence, e.g. both M+H[1+] and M(C13)+H[1+] forms present
- The summary overlap table must exist, be well-formed, and cover every cutoff value in the sweep
- A fraction of predicted metabolites should fall outside the original m/z annotation, as in the paper where 19 of 60 were not in the original annotation

## Limitations

- A single m/z feature may match several metabolites in the same module or pathway, inflating overlap counts between cutoffs
- Quantification and feature selection upstream of mummichog remain problematic and can confound the sensitivity assessment
- Current mummichog does not incorporate a statistical matrix from the feature selection step, so the sweep only captures cutoff-driven variation in the significant list
- Features from high-resolution profiling far exceed database annotations, causing de facto filtering that may exaggerate apparent robustness
- Metabolic models need improvement on lipid annotation, physiological context and tissue specificity, which affects which metabolites can be predicted at any cutoff

## Evidence

- [results] mummichog requires two lists of m/z features, the significant list and the reference list: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [supplementary] Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation.: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [results] Confidence rating system for metabolite prediction: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] Figure S7 shows cutoff sensitivity: "In the moDC data, we chose  by a cutoff value ."
- [results] Pathway analysis uses Fisher's exact test with EASE adjustment and Gamma-modeled permutation null: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
