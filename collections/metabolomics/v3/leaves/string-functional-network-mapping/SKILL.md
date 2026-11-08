---
name: string-functional-network-mapping
description: Use when per-gene enrichment scores (e.g., linear_msea_genebig_allpeaks_combined.csv)
  must be propagated over the STRING functional network to rank WES candidate genes
  for a single patient.
when_to_use_negative:
- When the patient's disease-associated metabolites are not measurable by the available
  metabolomic system (e.g., large glycosaminoglycans for the IDS patient), since ME
  scores for those genes will be missing or uninformative
- When curated metabolite identities are required rather than non-curated feature-level
  enrichment — TIDEX feature-to-metabolite mapping was unavailable, so exact metabolic
  identities cannot be determined
- When candidate lists are already short (trio WES with parental data), where the
  paper notes prioritization is already simplified and network smoothing adds less
  value
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0154
tools:
- name: metPropagate
  role: Provides the integration/label_propagation code that maps enrichment scores
    onto STRING node IDs and runs propagation
  repo: https://github.com/emmagraham/metPropagate
- name: STRING
  role: Functional linkage network (v11) over which ME/DME scores are assigned and
    propagated
- name: yamaguchiyuto/label_propagation
  role: Scikit-learn-compatible LGC label-propagation solver (alpha = 0.99) adapted
    by metPropagate
  repo: https://github.com/yamaguchiyuto/label_propagation
- name: Exomiser
  role: Supplies the per-patient candidate gene lists that are ranked by the propagated
    STRING scores
- name: HMDB
  role: Source of gene-metabolite associations used to compute the ME/DME enrichment
    scores fed into the network
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:2078eeac947a802f0e9a0ddac4afe104ff4bbc8d804ff00bdfa3db7e472c5b3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - Exomiser
  - HMDB
  - STRING
  - metPropagate
  - yamaguchiyuto/label_propagation
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# string-functional-network-mapping

## Summary

Maps per-gene metabolomic perturbation scores (ME/DME from Fisher's Exact enrichment against HMDB gene-metabolite sets) onto STRING v11 functional linkage network nodes, then propagates them with label propagation so candidate genes inherit evidence from network neighbors.

## When to use

Use when per-gene enrichment scores (e.g., linear_msea_genebig_allpeaks_combined.csv) must be propagated over the STRING functional network to rank WES candidate genes for a single patient.

## When NOT to use

- When the patient's disease-associated metabolites are not measurable by the available metabolomic system (e.g., large glycosaminoglycans for the IDS patient), since ME scores for those genes will be missing or uninformative
- When curated metabolite identities are required rather than non-curated feature-level enrichment — TIDEX feature-to-metabolite mapping was unavailable, so exact metabolic identities cannot be determined
- When candidate lists are already short (trio WES with parental data), where the paper notes prioritization is already simplified and network smoothing adds less value

## Inputs

- Per-patient metabolomic enrichment file (linear_msea_genebig_allpeaks_combined.csv) from the metPropagate METABOLOMIC_PROCESSING_PIPELINE
- STRING v11 functional linkage network (STRING_graph_file_v11_functional_entire_db.txt)
- STRING v11 gene-to-node_id mapping file (STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db)
- Per-gene label file of scaled ME scores (e.g., intermediate_label_file_sampleA)
- Exomiser candidate gene list per patient (e.g., sampleA_candidate_genes.csv, 281-609 genes)

## Outputs

- Post-propagation node score file (LPA_output) with Score.ID (propagated) and Original.Score.ID (original) per node
- Intermediate per-gene label file in integration/label_files (e.g., STRING_sampleA)
- Ranked candidate gene list enabling top-20th-percentile causative gene check (rank <= ~60 for a 300-gene list)

## How to apply

Take the per-patient metabolomic enrichment file (output/linear_msea_genebig_allpeaks_combined.csv combining positive and negative ESI mode DAM enrichment) and map gene names to STRING v11 node IDs using the STRING_graph_file_v11_gene_to_nodeid_mapping file. Scale each gene's DME/ME score between 0 and 1 to initialize labels y, then run label propagation over STRING_graph_file_v11_functional_entire_db using the Zhou et al. LGC formulation (alpha/lambda = 0.99) from the yamaguchiyuto/label_propagation implementation until convergence. Extract the propagated score (Score.ID) and original label (Original.Score.ID) for every node, subset to the patient's Exomiser candidate gene list (281-609 genes per TIDEX patient), and rank candidates by propagated score.

## Related tools

- **metPropagate** (Provides the integration/label_propagation code that maps enrichment scores onto STRING node IDs and runs propagation) — https://github.com/emmagraham/metPropagate
- **STRING** (Functional linkage network (v11) over which ME/DME scores are assigned and propagated)
- **yamaguchiyuto/label_propagation** (Scikit-learn-compatible LGC label-propagation solver (alpha = 0.99) adapted by metPropagate) — https://github.com/yamaguchiyuto/label_propagation
- **Exomiser** (Supplies the per-patient candidate gene lists that are ranked by the propagated STRING scores)
- **HMDB** (Source of gene-metabolite associations used to compute the ME/DME enrichment scores fed into the network)

## Examples

```
python main.py lgc -g STRING_graph_file_v11_functional_entire_db.txt -l intermediate_label_file_sampleA -o sampleA_output
```

## Evaluation signals

- Output file contains both Score.ID and Original.Score.ID for every candidate gene, allowing direct comparison of propagated vs pre-propagation ranks
- Initial labels y are scaled between 0 and 1 from the scaled DME scores before propagation
- LGC solver runs with alpha = 0.99 (Zhou et al. formulation) and converges over the full STRING v11 functional network
- At least one causative gene falls in the top 20th percentile of the ranked candidate list for 9/11 TIDEX patients (55-115 genes for 281-609-gene lists)
- Propagated ranking outperforms Exomiser's hiPHIVE phenotype-based ranking in 6/11 patients when benchmarked

## Limitations

- STRING v11 was chosen for benchmarking performance but the discussion reports no explicit software version justification beyond that choice
- Metabolites commonly used as IEM biomarkers (acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB, weakening ME scores for those genes
- Results reflect singleton exomes (281-609 candidate genes); trio WES would shorten lists and the reported top-20th-percentile counts are not directly transferable

## Evidence

- [intro] We then use this information to assign a score to each protein in a protein functional linkage network (STRING): "We then use this information to assign a score to each protein in a protein functional linkage network (STRING)"
- [results] DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [results] propagating per-gene ME scores across a protein–protein functional linkage network: "propagating per-gene ME scores across a protein–protein functional linkage network"
- [methods] adapted from an implementation found at https://github.com/yamaguchiyuto/label_propagation: "adapted from an implementation found at https://github.com/yamaguchiyuto/label_propagation"
- [readme] STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db: node_id to gene name mapping file.: "STRING_graph_file_v11_gene_to_nodeid_mapping_functional_entire_db: node_id to gene name mapping file."
