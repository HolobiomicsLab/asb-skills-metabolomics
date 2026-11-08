---
name: single-cell-differential-abundance-testing
description: Use when you have a per-cell metabolomics matrix (e.g. MORPHnMOL.csv)
  with condition metadata and need to rank features by abundance shift between two
  cell populations, such as inflammatory vs benign steatotic states.
when_to_use_negative:
- Input is already a differential abundance table or the populations have not been
  segmented into per-cell profiles — no per-cell matrix means no per-cell test
- The question is clustering/discovery of metabolic states rather than testing a pre-specified
  two-condition contrast; state discovery in the SpaceM NAFLD/NASH model used separate
  state-assignment analyses
- Data are bulk metabolomics without single-cell resolution — the article motivates
  single-cell measurements precisely because existing efforts 'lack single-cell measurements'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: SCANPY
  role: Runs the per-metabolite Wilcoxon rank-sum differential abundance test (rank_genes_groups
    with multiple-testing correction) on the per-cell matrix
- name: SpaceM
  role: Upstream method that produced the per-cell spatio-molecular matrix being tested
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Execution environment for loading MORPHnMOL.csv and orchestrating the test
    and CSV export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:846a63180c486bc58205ecd22175c560388c02ad82e759715f9a079a69b9adfc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-differential-abundance-testing

## Summary

Per-metabolite non-parametric differential abundance testing on SpaceM per-cell spatio-molecular data, e.g. comparing IL-17A-induced inflammatory steatotic hepatocytes against benign steatotic (+FA) cells to identify sphingomyelin/ceramide-phosphocholine enrichment.

## When to use

Use when you have a per-cell metabolomics matrix (e.g. MORPHnMOL.csv) with condition metadata and need to rank features by abundance shift between two cell populations, such as inflammatory vs benign steatotic states.

## When NOT to use

- Input is already a differential abundance table or the populations have not been segmented into per-cell profiles — no per-cell matrix means no per-cell test
- The question is clustering/discovery of metabolic states rather than testing a pre-specified two-condition contrast; state discovery in the SpaceM NAFLD/NASH model used separate state-assignment analyses
- Data are bulk metabolomics without single-cell resolution — the article motivates single-cell measurements precisely because existing efforts 'lack single-cell measurements'

## Inputs

- MORPHnMOL.csv — SpaceM per-cell spatio-molecular matrix (normalized metabolite intensities + morphology/fluorescence features)
- Culture-condition metadata columns annotating cells as +FA (benign steatotic) or +FA+IL17A (inflammatory steatotic)
- Metabolite annotation mapping (e.g. ceramide phosphocholine / sphingomyelin feature labels)

## Outputs

- differential_abundance_inflammatory_hepatocytes.csv — ranked differential abundance table with log fold-change, adjusted p-value, and metabolite annotation columns
- Sub-set summary of sphingomyelin/ceramide-phosphocholine enrichment (direction and significance)

## How to apply

Load the per-cell spatio-molecular matrix MORPHnMOL.csv (normalized metabolite intensities plus morphology/fluorescence features per cell) into Python with pandas, and split cells into the two populations (here +FA 'benign' steatotic vs +FA+IL17A 'inflammatory' steatotic hepatocytes) using the culture-condition metadata columns. Run a per-metabolite Wilcoxon rank-sum test via scanpy's rank_genes_groups with multiple-testing correction, treating the metabolite features like genes in a single-cell object. Subset and summarize results for sphingomyelin (ceramide phosphocholine) features to assess direction and significance of enrichment in the inflammatory state, mirroring Supp. Fig 12. Write the ranked table (log fold-change, adjusted p-value, metabolite annotations) to differential_abundance_inflammatory_hepatocytes.csv.

## Related tools

- **SCANPY** (Runs the per-metabolite Wilcoxon rank-sum differential abundance test (rank_genes_groups with multiple-testing correction) on the per-cell matrix)
- **SpaceM** (Upstream method that produced the per-cell spatio-molecular matrix being tested) — github.com/alexandrovteam/SpaceM
- **Python** (Execution environment for loading MORPHnMOL.csv and orchestrating the test and CSV export)

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, groupby='condition', groups=['FA_IL17A'], reference='FA', method='wilcoxon')
```

## Evaluation signals

- Output CSV exists and contains log fold-change, adjusted p-value, and metabolite annotation columns
- Sphingomyelin/ceramide-phosphocholine features show positive log fold-change with significant adjusted p-values in the +FA+IL17A inflammatory group
- Enrichment direction matches Supp. Fig 12 of the SpaceM manuscript
- Both contrast groups (+FA and +FA+IL17A) are non-empty after metadata-based splitting; per the source card, 93% of +FA+IL17A cells resided in the steatotic state versus 24.3% of +FA cells
- Multiple-testing correction has been applied so reported p-values are adjusted p-values, not raw

## Limitations

- Excluding co-sampling ablation marks did not change metabolic state markers but reduced their statistical significance, likely due to the lower number of cells
- Co-sampling of neighboring cells by the same ablation mark can occur, though it only minimally affected cell type prediction accuracy
- A mechanistic explanation for increased sphingolipids in NASH is still lacking — the test reports association (enrichment), not mechanism
- SpaceM tile-stitching code is optimized for the Nikon Ti-E microscope output format, which constrains upstream matrix generation

## Evidence
