---
name: multiple-testing-correction-statistics
description: Use when comparing two cell groups (e.g. +FA benign vs +FA+IL17A inflammatory
  steatotic hepatocytes) across hundreds of metabolites in a per-cell matrix and uncorrected
  p-values would inflate false positives.
when_to_use_negative:
- Input is already a differential table with adjusted p-values (correction already
  applied)
- The comparison is a single hypothesis with one p-value, where no family-wise correction
  across features is needed
- Cells have not been assigned to the states/conditions being compared (grouping must
  precede the per-feature tests)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: SCANPY
  role: Provides rank_genes_groups for Wilcoxon rank-sum testing with multiple-testing
    correction on the per-cell metabolite matrix
  repo: github.com/alexandrovteam/SpaceM
- name: SpaceM
  role: Generates the spatio-molecular matrix (MORPHnMOL.csv) on which the per-metabolite
    tests are run
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Execution environment for the scanpy-based differential abundance workflow
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:44177d57d06aa412c2e439a36159fa4c9310bacaea08dabb706aada96011a2fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multiple-testing-correction-statistics

## Summary

Statistical skill for applying multiple-testing correction to per-feature p-values from non-parametric group comparisons — here, Wilcoxon rank-sum differential abundance of metabolites between single-cell metabolic states in SpaceM spatio-molecular data.

## When to use

Use when comparing two cell groups (e.g. +FA benign vs +FA+IL17A inflammatory steatotic hepatocytes) across hundreds of metabolites in a per-cell matrix and uncorrected p-values would inflate false positives.

## When NOT to use

- Input is already a differential table with adjusted p-values (correction already applied)
- The comparison is a single hypothesis with one p-value, where no family-wise correction across features is needed
- Cells have not been assigned to the states/conditions being compared (grouping must precede the per-feature tests)

## Inputs

- Per-cell spatio-molecular matrix MORPHnMOL.csv (normalized metabolite intensities + morphology/fluorescence features)
- Cell-group labels from culture-condition metadata columns (e.g. +FA vs +FA+IL17A)

## Outputs

- Ranked differential abundance table (log fold-change, adjusted p-value, metabolite annotations), e.g. differential_abundance_inflammatory_hepatocytes.csv

## How to apply

Load the per-cell spatio-molecular matrix MORPHnMOL.csv (normalized metabolite intensities plus morphology/fluorescence features) and split cells by culture-condition metadata into the groups to compare. Run a per-metabolite non-parametric test — Wilcoxon rank-sum via scanpy's rank_genes_groups — so no normality assumption is placed on metabolite intensity distributions, then apply multiple-testing correction so adjusted p-values control the false-discovery rate across all tested metabolites. Rank metabolites by log fold-change and adjusted p-value and summarize the direction/significance of the feature class of interest (e.g. sphingomyelins/ceramide phosphocholines in the inflammatory state). Write the ranked table (log fold-change, adjusted p-value, metabolite annotation columns) to CSV. Note from the SpaceM hepatocyte study that statistical significance of state markers can drop when cell numbers shrink (e.g. after excluding co-sampling ablation marks), so correction interacts directly with sample size.

## Related tools

- **SCANPY** (Provides rank_genes_groups for Wilcoxon rank-sum testing with multiple-testing correction on the per-cell metabolite matrix) — github.com/alexandrovteam/SpaceM
- **SpaceM** (Generates the spatio-molecular matrix (MORPHnMOL.csv) on which the per-metabolite tests are run) — github.com/alexandrovteam/SpaceM
- **Python** (Execution environment for the scanpy-based differential abundance workflow) — github.com/alexandrovteam/SpaceM

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='condition', groups=['FA_IL17A'], reference='FA', method='wilcoxon', corr_method='benjamini-hochberg')
```

## Evaluation signals

- Output CSV exists and contains log fold-change, adjusted p-value, and metabolite annotation columns
- Sphingomyelin/ceramide-phosphocholine features show consistent enrichment (positive log fold-change with significant adjusted p-values) in the +FA+IL17A inflammatory group, matching Supp. Fig 12 direction
- Adjusted p-values are >= raw p-values for every tested metabolite (correction only increases them)
- Group balance is biologically plausible — e.g. the SpaceM study reports 93% of +FA+IL17A hepatocytes in the steatotic state versus 24.3% among +FA cells
- State-marker significance degrades predictably with reduced cell counts (as observed when excluding co-sampling ablation marks)

## Limitations

- Excluding co-sampling ablation marks reduces the number of cells and can reduce statistical significance of metabolic state markers without changing which markers are found
- Intermediate-state cell bimodality could reflect a clustering artifact rather than biology, complicating group definitions before testing
- No established validation approaches currently exist for single-cell metabolomics, limiting independent verification of hit lists

## Evidence

- [other] Run a per-metabolite non-parametric differential abundance test (Wilcoxon rank-sum via scanpy's rank_genes_groups, with multiple-testing correction): [evidence span withheld — anchor status: not_found]
