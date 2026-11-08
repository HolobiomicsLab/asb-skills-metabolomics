---
name: sphingomyelin-metabolite-enrichment-interpretation
description: Use when a SpaceM per-cell matrix (MORPHnMOL.csv) must be split by culture
  condition (e.g. +FA vs +FA+IL17A) and specific lipid classes such as sphingomyelins
  assessed for direction/significance of enrichment between metabolic states.
when_to_use_negative:
- Input is already an aggregated/bulk metabolomics table with no per-cell spatio-molecular
  matrix — the Wilcoxon test requires single-cell intensity distributions per condition
- Cell culture-condition metadata columns are absent, so +FA vs +FA+IL17A populations
  cannot be annotated and split
- The goal is to establish a mechanism for sphingolipid increases — the paper explicitly
  notes a mechanistic explanation is still lacking
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SpaceM
  role: Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the
    input to the differential abundance test
  repo: https://github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Provides rank_genes_groups (Wilcoxon rank-sum with multiple-testing correction)
    used for per-metabolite differential abundance testing
- name: Python
  role: Pandas-based loading of MORPHnMOL.csv, condition annotation/splitting, result
    subsetting and CSV export
- name: METASPACE
  role: Upstream annotation of MALDI-imaging metabolites used to identify sphingomyelin
    features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:c117f1ca3e1c17601eaf7d8d352db2f58149b6e59fa46d0f748450f9a425573f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - METASPACE
  - Python
  - SCANPY
  - SpaceM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sphingomyelin-metabolite-enrichment-interpretation

## Summary

Tests whether sphingomyelin (ceramide phosphocholine) metabolite features are enriched in a specific single-cell metabolic state — here the IL-17A-induced inflammatory steatotic state versus benign fatty-acid-stimulated hepatocytes — using per-metabolite non-parametric differential abundance testing on SpaceM spatio-molecular data.

## When to use

Use when a SpaceM per-cell matrix (MORPHnMOL.csv) must be split by culture condition (e.g. +FA vs +FA+IL17A) and specific lipid classes such as sphingomyelins assessed for direction/significance of enrichment between metabolic states.

## When NOT to use

- Input is already an aggregated/bulk metabolomics table with no per-cell spatio-molecular matrix — the Wilcoxon test requires single-cell intensity distributions per condition
- Cell culture-condition metadata columns are absent, so +FA vs +FA+IL17A populations cannot be annotated and split
- The goal is to establish a mechanism for sphingolipid increases — the paper explicitly notes a mechanistic explanation is still lacking

## Inputs

- MORPHnMOL.csv — SpaceM per-cell spatio-molecular matrix with normalized metabolite intensities and morphology/fluorescence features
- Culture-condition metadata columns distinguishing +FA and +FA+IL17A cells
- Metabolite annotation mapping identifying ceramide phosphocholine (sphingomyelin) features

## Outputs

- Ranked differential abundance table (log fold-change, adjusted p-value, metabolite annotations) per metabolite
- differential_abundance_inflammatory_hepatocytes.csv
- Summary of sphingomyelin/ceramide-phosphocholine enrichment direction and significance in the +FA+IL17A inflammatory state

## How to apply

Load the per-cell spatio-molecular matrix MORPHnMOL.csv (normalized metabolite intensities plus morphology/fluorescence features) with pandas and split cells into +FA ('benign' steatotic) and +FA+IL17A ('inflammatory' steatotic) populations via their culture-condition metadata columns. Run a per-metabolite Wilcoxon rank-sum differential abundance test via scanpy's rank_genes_groups with multiple-testing correction, comparing inflammatory vs benign cells. Subset the ranked results to ceramide phosphocholine (sphingomyelin) features and interpret their enrichment as positive log fold-change with significant adjusted p-values in the inflammatory group, mirroring Supp. Fig 12. Write the ranked table (log fold-change, adjusted p-value, metabolite annotations) to differential_abundance_inflammatory_hepatocytes.csv. Interpretation is biological, not mechanistic: the SpaceM paper notes that increased sphingolipids in the inflammatory state are consistent with oxidative stress and systemic inflammation but lack a mechanistic explanation, so report enrichment direction and significance rather than claiming causality.

## Related tools

- **SpaceM** (Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the input to the differential abundance test) — https://github.com/alexandrovteam/SpaceM
- **SCANPY** (Provides rank_genes_groups (Wilcoxon rank-sum with multiple-testing correction) used for per-metabolite differential abundance testing)
- **Python** (Pandas-based loading of MORPHnMOL.csv, condition annotation/splitting, result subsetting and CSV export)
- **METASPACE** (Upstream annotation of MALDI-imaging metabolites used to identify sphingomyelin features)

## Examples

```
import scanpy as sc; adata = sc.read_csv('MORPHnMOL.csv').T; sc.tl.rank_genes_groups(adata, 'condition', groups=['FA_IL17A'], reference='FA', method='wilcoxon')
```

## Evaluation signals

- Output CSV exists with required columns: metabolite annotation, log fold-change, adjusted p-value
- Sphingomyelin/ceramide-phosphocholine features show consistent enrichment — positive log fold-change with significant adjusted p-values — in the +FA+IL17A group, matching Supp. Fig 12
- Wilcoxon rank-sum (non-parametric) testing with multiple-testing correction was applied, not a parametric t-test
- Population split is correct: in the source experiment 93% of +FA+IL17A hepatocytes were in the steatotic state vs 24.3% of +FA cells, so condition annotation must reproduce this known state balance

## Limitations

- Increased sphingolipids in the inflammatory/NASH state lack a mechanistic explanation — the test supports enrichment claims, not causal biology
- Excluding co-sampling ablation marks reduced statistical significance of metabolic state markers, likely due to the lower number of cells — sphingomyelin significance can weaken after such filtering
- SpaceM implementation requires stitched microscopy tiles and is optimized for Nikon Ti-E output format; data from other microscopes may need adaptation before the matrix is available

## Evidence

- [supplementary] sphingomyelin enrichment test on condition-split cells: [evidence span withheld — anchor status: not_found]
- [results] expected state balance validating condition annotation: [evidence span withheld — anchor status: not_found]
