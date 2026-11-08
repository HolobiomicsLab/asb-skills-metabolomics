---
name: protein-interaction-network-seeding
description: Use when a patient's differentially abundant metabolites (DAMs) fail
  to directly enrich the causative gene's metabolite set, and you need network propagation
  over protein-protein interactions to recover candidate genes from metabolomic enrichment
  scores.
when_to_use_negative:
- The candidate gene already has direct metabolomic evidence — if the causative gene's
  own HMDB-annotated metabolites are differentially abundant, ME-score ranking alone
  may suffice without network propagation.
- The metabolites driving the phenotype are unmeasured or unannotated (e.g. large
  glycosaminoglycans in the IDS patient, or acyl glycines/acyl carnitines not annotated
  to IEM genes in HMDB) — propagation cannot create signal absent from the seed labels.
- A physical-interaction-only network is required, since functional edges (e.g. co-citation
  via KEGG pathway membership) carry much of the propagated signal.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3520
tools:
- name: metPropagate
  role: 'Implements the seeding and label-propagation workflow: builds per-gene labels
    from ME scores and propagates them across the STRING network'
  repo: https://github.com/emmagraham/metPropagate
- name: STRING
  role: Supplies the protein functional linkage network (v11, 19,354 genes, physical
    + functional interactions) and node_id-to-gene mapping
- name: label_propagation (yamaguchiyuto)
  role: Scikit-learn-compatible LGC/Zhou et al. label-propagation solver adapted by
    metPropagate with lambda/alpha = 0.99
  repo: https://github.com/yamaguchiyuto/label_propagation
- name: HMDB
  role: Source of gene-metabolite associations used to define each gene's metabolite
    set for ME-score seeding
- name: numpy
  role: Python dependency for the label-propagation computation
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:7106dcf9ac0be4ac8084bc70cb7e4c96f4b027c7731cfef5b7b88d1771efb857
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - HMDB
  - NumPy
  - STRING
  - label_propagation (yamaguchiyuto)
  - metPropagate
  - numpy
  - scikit-learn
  - tidyverse
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# protein-interaction-network-seeding

## Summary

Seeds a protein functional linkage network (STRING v11, 19,354 genes) with per-gene metabolomic enrichment (ME/DME) scores and propagates them via label propagation to prioritize disease genes whose own metabolites are not differentially abundant.

## When to use

Use when a patient's differentially abundant metabolites (DAMs) fail to directly enrich the causative gene's metabolite set, and you need network propagation over protein-protein interactions to recover candidate genes from metabolomic enrichment scores.

## When NOT to use

- The candidate gene already has direct metabolomic evidence — if the causative gene's own HMDB-annotated metabolites are differentially abundant, ME-score ranking alone may suffice without network propagation.
- The metabolites driving the phenotype are unmeasured or unannotated (e.g. large glycosaminoglycans in the IDS patient, or acyl glycines/acyl carnitines not annotated to IEM genes in HMDB) — propagation cannot create signal absent from the seed labels.
- A physical-interaction-only network is required, since functional edges (e.g. co-citation via KEGG pathway membership) carry much of the propagated signal.

## Inputs

- Per-gene metabolomic enrichment (ME) file with scaled scores in [0,1] (e.g. linear_msea_genebig_allpeaks_combined.csv from the metabolomic processing pipeline)
- STRING v11 functional linkage network edge list (STRING_graph_file_v11_functional_entire_db.txt) and node_id-to-gene-name mapping file
- Candidate gene list per patient (e.g. Exomiser-filtered sampleA_candidate_genes.csv)
- Patient metabolite z-score matrix (e.g. 436 plasma metabolite z-scores for 107 Miller patients vs 70 controls)

## Outputs

- Label propagation output file listing each candidate gene's propagated score (Score.ID) and original score (Original.Score.ID) in LPA_output/results
- Per-patient prioritized candidate gene list ranked by propagated score
- Per-gene label file used as input to propagation (intermediate_label_file_<sample>)

## How to apply

Compute per-gene ME scores from Fisher's Exact Test enrichment of the patient's DAMs (|z-score| >= 2) among HMDB-annotated gene metabolite sets, scaled to [0,1] and multiplied by the gene's largest absolute metabolite z-score. Load the complete STRING v11 homo sapiens functional linkage network (physical plus functional interactions — full-network ranking beat physical-only in ~90% of causative genes), symmetrize edge weights, and set each gene's scaled ME score as its initial label. Run label propagation as formulated by Zhou et al. (f_i^(r) = lambda * sum_j w_ij f_j^(r-1) + (1-lambda) y_i) with lambda = 0.99, iterated until convergence over all 19,354 nodes. Rank candidate genes by the final propagated score (Score.ID) alongside the pre-propagation score (Original.Score.ID) to produce a per-patient prioritized gene list.

## Related tools

- **metPropagate** (Implements the seeding and label-propagation workflow: builds per-gene labels from ME scores and propagates them across the STRING network) — https://github.com/emmagraham/metPropagate
- **STRING** (Supplies the protein functional linkage network (v11, 19,354 genes, physical + functional interactions) and node_id-to-gene mapping)
- **label_propagation (yamaguchiyuto)** (Scikit-learn-compatible LGC/Zhou et al. label-propagation solver adapted by metPropagate with lambda/alpha = 0.99) — https://github.com/yamaguchiyuto/label_propagation
- **HMDB** (Source of gene-metabolite associations used to define each gene's metabolite set for ME-score seeding)
- **numpy** (Python dependency for the label-propagation computation)

## Examples

```
python main.py lgc -g STRING_graph_file_v11_functional_entire_db.txt -l intermediate_label_file_sampleA -o LPA_output/sampleA
```

## Evaluation signals

- Each propagated score (Score.ID) lies within the [0,1] label range and every candidate gene appears in the output file alongside its Original.Score.ID.
- On the Miller benchmark (107 patients, 21 IEMs), at least one causative gene falls in the top 20th percentile (rank <= 60 of 300 permutation genes) for ~91-92% of patients.
- Among patients whose causative genes lacked any ME score, propagation should still place a causative gene in the top 20th percentile for ~79% (33/42) of patients.
- Ranking under the complete physical+functional STRING network should beat the physical-only subset for ~90% of causative genes.
- Results in integration/LPA_output should reproduce after running run_met_processing.sh then run_label_prop_cluster.sh on the dummy sampleA inputs.

## Limitations

- Functional edges in STRING include co-citation links, so propagated signal can partly reflect literature bias rather than physical interaction.
- Diseases whose biomarkers are unmeasured by the metabolomic platform or unannotated to IEM genes in HMDB (e.g. acyl glycines, acyl carnitines) yield no seed signal for the causative gene.
- Lambda = 0.99 was taken from the Zhou et al. formulation and was not optimized.
- Label propagation requires the graph file to be unzipped (STRING_graph_file_v11_functional_entire_db.txt.zip) and the pipeline scripts call .pbs batch files that must be adapted (e.g. to SLURM).

## Evidence

- [results] Apply label propagation as formulated by Zhou et al. (f_i^(r) = lambda * sum_j w_ij f_j^(r-1) + (1 - lambda) y_i), using an iterative solver adapted from the yamaguchiyuto/label_propagation implementation, with lambda set to 0.99.: [evidence span withheld — anchor status: not_found]
- [results] DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [results] propagating per-gene ME scores across a protein–protein functional linkage network: "propagating per-gene ME scores across a protein–protein functional linkage network"
- [readme] Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID".: "final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID""
- [results] STRING v11 human functional linkage network (19,354 genes; both physical and functional interactions, which improved causative-gene ranking for 90% of cases versus physical-only): [evidence span withheld — anchor status: not_found]
- [readme] This section contains code for the label propagation component of metPropagate. It was adapted from Yuto Yamaguchi (https://github.com/yamaguchiyuto/label_propagation).: "This section contains code for the label propagation component of metPropagate. It was adapted from Yuto Yamaguchi (https://github.com/yamaguchiyuto/label_propagation)."
