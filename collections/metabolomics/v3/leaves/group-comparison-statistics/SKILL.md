---
name: group-comparison-statistics
description: Use when comparing a continuous measurement (e.g., metabolite concentration
  in µg/g tissue) between two small, independent biological subgroups (e.g., phosphoAKT1-high/MYC-low
  n=14 vs. phosphoAKT1-low/MYC-high n=5) without assuming normality.
when_to_use_negative:
- When the data are high-dimensional expression or spectral intensities from the untargeted
  screen (hundreds of features) where a multivariate GSEA or k-means clustering approach
  is more appropriate
- When the comparison involves more than two groups or requires correction for multiple
  testing across all metabolites simultaneously without prior biological selection
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GraphPad Prism
  role: Statistical software used to run the Mann-Whitney Test for pairwise comparison
    of absolute metabolite concentrations between tumor subgroups
- name: Python
  role: Scripting environment for parsing metabolite tables, assembling comparison
    groups, and formatting statistical results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:0661de8b109bff8583bf9aa0ff22ea7f7bf3808b33d3c6618ef882a21efcd125
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - Python
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# group-comparison-statistics

## Summary

Applies non-parametric hypothesis testing to compare metabolite concentrations between molecularly defined tumor subgroups, as used in the Priolo et al. 2014 study to validate untargeted metabolomics findings with absolute quantitation.

## When to use

Use when comparing a continuous measurement (e.g., metabolite concentration in µg/g tissue) between two small, independent biological subgroups (e.g., phosphoAKT1-high/MYC-low n=14 vs. phosphoAKT1-low/MYC-high n=5) without assuming normality.

## When NOT to use

- When the data are high-dimensional expression or spectral intensities from the untargeted screen (hundreds of features) where a multivariate GSEA or k-means clustering approach is more appropriate
- When the comparison involves more than two groups or requires correction for multiple testing across all metabolites simultaneously without prior biological selection

## Inputs

- Absolute concentration values (µg/g tissue) for selected metabolites measured by targeted LC-MS/MS, grouped by tumor subtype
- Sample-size table mapping each biological specimen to its phosphoAKT1/MYC subgroup (e.g., 40 tumors, 16 normal samples)
- List of candidate metabolites to test (e.g., oleic, arachidonic, docosahexaenoic acids, creatine, 2-aminoadipic acid)

## Outputs

- Per-metabolite Mann-Whitney test result with p-value and direction of change (e.g., 'oleic acid significantly higher in phosphoAKT1-low/MYC-high tumors')
- Validation table matching each metabolite to the untargeted metabolomics signal it corroborates or refutes
- Statistical summary used to support the finding that AKT1/MYC status drives distinct metabolic fingerprints

## How to apply

Extract the absolute concentrations of each target metabolite (oleic acid, arachidonic acid, docosahexaenoic acid, creatine, 2-aminoadipic acid) from the untargeted metabolomics dataset, then organize them by the k-means-defined phosphoAKT1/MYC tumor subgroup. For each metabolite, run a Mann-Whitney Test (two-tailed) in GraphPad Prism comparing the two groups of interest, such as phosphoAKT1-low/MYC-high (n=5) vs. phosphoAKT1-high/MYC-low (n=14) tumors. Report the direction of the fold-change and the exact p-value, and call the metabolite significantly different when p < 0.05, which was the threshold used to validate the untargeted lipid-metabolism and glycolysis signals. Because sample sizes are small and unbalanced, avoid parametric t-tests; the non-parametric rank-based test is the appropriate choice.

## Related tools

- **GraphPad Prism** (Statistical software used to run the Mann-Whitney Test for pairwise comparison of absolute metabolite concentrations between tumor subgroups)
- **Python** (Scripting environment for parsing metabolite tables, assembling comparison groups, and formatting statistical results)

## Examples

```
python scripts/mann_whitney_validation.py --metabolites oleic,arachidonic,dha --group_a phosphoAKT1-low_MYC-high --group_b phosphoAKT1-high_MYC-low --alpha 0.05
```

## Evaluation signals

- Every p-value reported is < 0.05 for a metabolite claimed to be significant, and the direction of change matches the sign of the group medians.
- The n used in the test equals the count of specimens in each subgroup stated in the sample table (e.g., n=5 vs n=14), not the total dataset size.
- The test used is explicitly a Mann-Whitney (non-parametric) test, not a Student's t-test.
- Each validated metabolite (oleic, arachidonic, DHA, creatine, 2-aminoadipic acid) can be traced back to a specific claim in the Results And Discussion section.
- Units are consistently µg/g tissue and the comparison groups are phosphoAKT1/MYC-defined, not based on Gleason grade or stage.

## Limitations

- Sample sizes are small and unbalanced (e.g., 5 vs 14 tumors), reducing statistical power; results should be interpreted as validation of untargeted signals rather than definitive effect-size estimates.
- The p < 0.05 threshold was applied without reporting an explicit multiple-testing correction across the panel of five metabolites.
- The test is only valid for two independent groups; it does not handle the full 4-subgroup k-means clustering in a single analysis.

## Evidence

- [methods] All three validated fatty acids (oleic, arachidonic, and docosahexaenoic acids) were present at significantly higher concentrations in MYC-high tumors than in phosphoAKT1-high/MYC-low tumors.: [evidence span withheld — anchor status: not_found]
- [methods] the use of the Mann-Whitney Test, and the p<0.05 significance threshold: [evidence span withheld — anchor status: not_found]
