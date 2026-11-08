---
name: graph-edge-weight-normalization
description: Use when a STRING functional linkage graph or other directed weighted
  interaction network must be converted into a symmetric weighted adjacency/edge list
  before label propagation of per-gene metabolomic enrichment scores.
when_to_use_negative:
- The graph is already an undirected, symmetric edgelist in the label_propagation
  solver's two-column format — re-symmetrizing is redundant work.
- Input is still raw LC/MS feature data (XCMS positive/negative intensity matrices,
  HMDB feature tables); no interaction network exists yet and metabolomic enrichment
  must be computed first.
- The goal is variant-level candidate generation (Exomiser variant effect and population
  frequency filters on DeepVariant singleton VCFs), which precedes network propagation
  and does not involve edge weights.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3308
tools:
- name: metPropagate
  role: Integration stage that symmetrizes STRING edge weights, generates label files
    from metabolomic enrichment, and runs propagation to output Score.ID/Original.Score.ID
    per candidate gene
  repo: github.com/emmagraham/metPropagate
- name: STRING
  role: Source of the v11 protein functional linkage network whose weighted interactions
    are symmetrized into the propagation graph
  repo: stringdb.org
- name: label_propagation (LGC)
  role: Provides the scikit-learn-compatible Local and Global Consistency solver (alpha
    default 0.99) that consumes the normalized edgelist and label file
  repo: https://github.com/yamaguchiyuto/label_propagation
- name: scikit-learn
  role: Interface target of the LGC implementation (fit/predict estimator), enabling
    grid search and cross-validation over the normalized graph and labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:8b4d3d481f2677ea14507b80f2d6d4c8952a65e005e0df5228f358503e36fd22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - STRING
  - label_propagation (LGC)
  - metPropagate
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# graph-edge-weight-normalization

## Summary

Symmetrizes the weighted STRING v11 homo sapiens functional linkage network into an undirected edge list ('[src node id] [dst node id]') so per-gene metabolomic enrichment (ME/DME) labels scaled to [0,1] can be propagated by the Zhou et al. LGC solver with lambda/alpha = 0.99.

## When to use

Use when a STRING functional linkage graph or other directed weighted interaction network must be converted into a symmetric weighted adjacency/edge list before label propagation of per-gene metabolomic enrichment scores.

## When NOT to use

- The graph is already an undirected, symmetric edgelist in the label_propagation solver's two-column format — re-symmetrizing is redundant work.
- Input is still raw LC/MS feature data (XCMS positive/negative intensity matrices, HMDB feature tables); no interaction network exists yet and metabolomic enrichment must be computed first.
- The goal is variant-level candidate generation (Exomiser variant effect and population frequency filters on DeepVariant singleton VCFs), which precedes network propagation and does not involve edge weights.

## Inputs

- STRING v11 homo sapiens functional linkage network, entire database (integration/graph_files/STRING_graph_file_v11_functional_entire_db.txt.zip, unzipped before use)
- node_id-to-gene-name mapping file (integration/graph_files/STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db)
- Per-sample metabolomic enrichment scores (e.g., linear_msea_genebig_allpeaks_combined.csv from METABOLOMIC_PROCESSING_PIPELINE) scaled to [0,1] as initial continuous labels y

## Outputs

- Symmetrized weighted edgelist in '[src node id] [dst node id]' format (sample.edgelist style) consumed by main.py lgc
- Intermediate label file (e.g., intermediate_label_file_sampleA) mapping STRING node ids to [0,1] metabolomic enrichment labels
- LPA_output results file listing each candidate gene's propagated Score.ID alongside its pre-propagation Original.Score.ID, ranked by propagated score

## How to apply

Unzip STRING_graph_file_v11_functional_entire_db.txt (the STRING v11 entire-database functional interaction graph downloaded from stringdb.org) and pair it with STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db so every node id can be translated back to a gene name. Symmetrize the graph by emitting each STRING interaction in both directions with the same weight, because the propagation update f_i^(r) = lambda * sum_j w_ij f_j^(r-1) + (1 - lambda) y_i requires sum_j w_ij to cover every neighbor of gene i symmetrically. Write the symmetrized graph as a two-column integer edgelist in the sample.edgelist format consumed by the adapted yamaguchiyuto/label_propagation code, and build the label file from the combined metabolomic enrichment output (linear_msea_genebig_allpeaks_combined.csv) with DME scores scaled between 0 and 1. Run LGC with alpha = lambda = 0.99 (the Zhou et al. Local and Global Consistency default) iterating f^(r) over all nodes until convergence, then rank candidate genes by the final propagated Score.ID while preserving Original.Score.ID for pre-propagation comparison.

## Related tools

- **metPropagate** (Integration stage that symmetrizes STRING edge weights, generates label files from metabolomic enrichment, and runs propagation to output Score.ID/Original.Score.ID per candidate gene) — github.com/emmagraham/metPropagate
- **STRING** (Source of the v11 protein functional linkage network whose weighted interactions are symmetrized into the propagation graph) — stringdb.org
- **label_propagation (LGC)** (Provides the scikit-learn-compatible Local and Global Consistency solver (alpha default 0.99) that consumes the normalized edgelist and label file) — https://github.com/yamaguchiyuto/label_propagation
- **scikit-learn** (Interface target of the LGC implementation (fit/predict estimator), enabling grid search and cross-validation over the normalized graph and labels)

## Examples

```
python main.py lgc -g STRING_graph_file_v11_functional_entire_db.txt -l intermediate_label_file_sampleA -o LPA_output/sampleA
```

## Evaluation signals

- Edgelist is strictly two integer columns per line ('[src node id] [dst node id]'), matching the sample.edgelist format accepted by main.py lgc.
- Every STRING interaction appears in both directions (i,j) and (j,i) with equal weight, so sum_j w_ij covers all neighbors of every gene symmetrically.
- All final Score.ID values fall within the [0,1] label range, and each gene's Original.Score.ID is retained for pre-propagation comparison.
- Every propagated node id resolves to a gene name via STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db.
- Propagation ran with alpha/lambda = 0.99 (the Zhou et al. LGC default), and candidate genes are ranked by the resulting propagated score.

## Limitations

- The STRING entire-db graph ships zipped (STRING_graph_file_v11_functional_entire_db.txt.zip) and must be unzipped before running, per the README quick start.
- The paper reports no STRING database/software version, so reproducing exact edge weights depends on the manually placed v11 files in integration/graph_files.
- Symmetrized functional linkage assumes neighbor similarity, so genes whose metabolites are undetectable on the platform (e.g., large glycosaminoglycans for the IDS patient) are ranked purely by propagated neighbor signal.
- The yamaguchiyuto LGC implementation shows max_iter=30 in its example fit; convergence of propagated scores over the full STRING node set should be verified.

## Evidence

- [results] Per-gene weights on STRING are scaled to [0,1]: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [intro] Scores assigned on STRING then propagated across the network: "We then use this information to assign a score to each protein in a protein functional linkage network (STRING), and then propagate this evidence across the network"
- [methods] Propagation solver provenance: "adapted from an implementation found at https://github.com/yamaguchiyuto/label_propagation"
- [readme] node_id-to-gene mapping is a graph input: "STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db: node_id to gene name mapping file."
- [readme] LGC hyperparameter default lambda/alpha = 0.99: [evidence span withheld — anchor status: external_doc]
- [readme] Output schema Score.ID and Original.Score.ID: "File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID"."
