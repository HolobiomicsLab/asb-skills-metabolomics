---
name: network-label-propagation
description: Use when a patient-level per-gene score vector (e.g., metabolomic enrichment)
  needs to be smoothed/ranked over a protein interaction network so that neighborhood
  signal can implicate genes lacking direct annotation.
when_to_use_negative:
- When the analysis goal is ranking genes by direct gene-metabolite annotation alone
  — the ME score alone prioritized 31% fewer Miller patients than propagation across
  the STRING network
- When the candidate list must be restricted to physically interacting proteins only
  — the physical-only STRING subset ranked causative genes worse for ~90% of cases
  and omitted several causative genes entirely
- When no gene-metabolite association database (HMDB) or patient-vs-control metabolomic
  intensity matrix exists to construct the initial label vector
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0622
tools:
- name: metPropagate
  role: Implements the metabolomic-enrichment seeding of the STRING network and the
    label-propagation prioritization of candidate genes
  repo: github.com/emmagraham/metPropagate
- name: STRING
  role: Supplies the protein functional linkage network (v11, 19,354 genes, physical
    plus functional edges) over which labels are propagated
- name: yamaguchiyuto/label_propagation
  role: Provides the scikit-learn compatible iterative label-propagation solver (LGC-style)
    adapted for the propagation step
  repo: https://github.com/yamaguchiyuto/label_propagation
- name: Exomiser
  role: Produces the per-patient candidate gene lists (281-609 genes) that are ranked
    by propagated scores
- name: numpy
  role: Python dependency for the label-propagation solver
- name: scikit-learn
  role: Python dependency for the label-propagation solver interface
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:69648425d677935bda28875c28308825ce8e6fe657e9779a59ac7531edd89052
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - HMDB
  - NumPy
  - STRING
  - metPropagate
  - numpy
  - scikit-learn
  - tidyverse
  - yamaguchiyuto/label_propagation
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# network-label-propagation

## Summary

Seed a protein functional linkage network (STRING) with per-gene metabolomic enrichment (ME/DME) scores and propagate the labels with the Zhou et al. label-propagation formulation to rank candidate disease genes per patient, as implemented in metPropagate.

## When to use

Use when a patient-level per-gene score vector (e.g., metabolomic enrichment) needs to be smoothed/ranked over a protein interaction network so that neighborhood signal can implicate genes lacking direct annotation.

## When NOT to use

- When the analysis goal is ranking genes by direct gene-metabolite annotation alone — the ME score alone prioritized 31% fewer Miller patients than propagation across the STRING network
- When the candidate list must be restricted to physically interacting proteins only — the physical-only STRING subset ranked causative genes worse for ~90% of cases and omitted several causative genes entirely
- When no gene-metabolite association database (HMDB) or patient-vs-control metabolomic intensity matrix exists to construct the initial label vector

## Inputs

- Per-patient metabolomic enrichment file (linear_msea_genebig_allpeaks_combined.csv) with per-gene ME/DME scores
- STRING v11 homo sapiens functional linkage network (STRING_graph_file_v11_functional_entire_db), physical plus functional interactions
- STRING node_id-to-gene-name mapping file (STRING_graph_file_v11_gene_to_nodeid_mapping)
- Per-patient candidate gene list from Exomiser variant filtering (e.g., sampleA_candidate_genes.csv)
- Label propagation parameters: lambda = 0.99, labels scaled to [0,1]

## Outputs

- Per-patient propagated score file in LPA_output/results with Score.ID (post-propagation) and Original.Score.ID (pre-propagation) for every candidate gene
- Ranked prioritized candidate gene list per patient
- Summary table of per-patient rankings and top-20th-percentile prioritization outcomes (e.g., miller_metpropagate_rankings_summary.csv)

## How to apply

Map per-gene ME/DME scores onto STRING v11 node IDs via the gene-to-nodeid mapping file, scale scores to continuous labels y in [0,1], and symmetrize edge weights of the full STRING functional linkage network (physical plus functional interactions, 19,354 genes). Run label propagation with the Zhou et al. update f_i^(r) = lambda * sum_j w_ij f_j^(r-1) + (1 - lambda) y_i using the iterative solver adapted from yamaguchiyuto/label_propagation (LGC-style), with lambda = 0.99 (not optimized), iterating until convergence over all nodes. Rank each patient's Exomiser-derived candidate genes (e.g., 281–609 genes for TIDEX patients) by the propagated Score.ID and compare against the pre-propagation Original.Score.ID to generate the prioritized gene list.

## Related tools

- **metPropagate** (Implements the metabolomic-enrichment seeding of the STRING network and the label-propagation prioritization of candidate genes) — github.com/emmagraham/metPropagate
- **STRING** (Supplies the protein functional linkage network (v11, 19,354 genes, physical plus functional edges) over which labels are propagated)
- **yamaguchiyuto/label_propagation** (Provides the scikit-learn compatible iterative label-propagation solver (LGC-style) adapted for the propagation step) — https://github.com/yamaguchiyuto/label_propagation
- **Exomiser** (Produces the per-patient candidate gene lists (281-609 genes) that are ranked by propagated scores)
- **numpy** (Python dependency for the label-propagation solver)
- **scikit-learn** (Python dependency for the label-propagation solver interface)

## Examples

```
python main.py lgc -g STRING_graph_file_v11_functional_entire_db.edgelist -l intermediate_label_file_sampleA.label -o sampleA_output --alpha 0.99
```

## Evaluation signals

- Output file lists final propagated scores for every candidate gene and each propagated score lies within the [0,1] label range
- Candidate genes are ranked by propagated score, and at least one causative gene falls in the top 20th percentile (rank <= 60 of 300) for ~92% of Miller patients (98/107)
- Full-STRING vs physical-only-STRING ablation: ~90% of causative genes rank better on the complete physical + functional network, with several causative genes missing from the physical-only network
- For patients whose causative genes lacked an ME score, propagation still places a causative gene in the top 20th percentile for 79% (33/42) of patients
- Permutation benchmark (1000 random 300-gene lists) yields median causative-gene ranks consistent with Supplementary Data Set 1

## Limitations

- Lambda = 0.99 was used as in Zhou et al. and was not optimized for this task
- Available chromatographic/MS technologies cannot measure all metabolites, so some disease-associated metabolites are missed and the analysis is insensitive to diseases tied to unmeasured metabolites (e.g., large glycosaminoglycans for the IDS patient)
- Metabolites commonly used as IEM biomarkers (acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB
- No software version was reported for the STRING network or Exomiser/hiPHIVE

## Evidence

- [methods] Zhou label propagation with lambda=0.99 over STRING network: [evidence span withheld — anchor status: not_found]
- [results] Full STRING network outperforms physical-only subset: [evidence span withheld — anchor status: not_found]
- [results] Propagation recovers genes without direct ME scores: [evidence span withheld — anchor status: not_found]
- [readme] Output schema of propagated scores: "Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as"
- [results] DME scores as network weights: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
