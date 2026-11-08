---
name: network-neighborhood-metabolic-annotation
description: Use when per-patient metPropagate LPA propagated scores and Exomiser-Phenotype
  candidate scores (Exomiser-filtered lists of ~281-609 genes) must be merged to boost
  causative-gene ranks for suspected IEM patients.
when_to_use_negative:
- No protein functional linkage network or gene-metabolite annotations are available
  (e.g., non-human organism or no STRING/HMDB coverage), so the per-gene p weight
  cannot be computed
- Input is raw LC/MS feature data requiring the XCMS/metabolomic-processing pipeline
  rather than already-prioritized candidate gene lists from Exomiser filtering
- Candidate lists are already very short (e.g., trio WES with parental data), so the
  top-20th-percentile binning and reweighting add little value
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: metPropagate
  role: Supplies the propagated per-gene metabolomic scores (LPA output) that form
    the (1-p) weighted term of the combination
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Supplies the phenotype-based candidate gene scores and variant-filtered candidate
    lists that form the p weighted term
- name: STRING
  role: Provides the v11 protein functional linkage network used to compute first-degree-neighbor
    HMDB annotation percentages
- name: yamaguchiyuto/label_propagation
  role: Label-propagation implementation adapted by metPropagate to propagate metabolomic
    enrichment scores across the STRING network
  repo: https://github.com/yamaguchiyuto/label_propagation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:f54c58b3272ddd4b090d4b800cee0a8b1e7cf0d607a0d7e499f060f7d43a21ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - STRING
  - metPropagate
  - yamaguchiyuto/label_propagation
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# network-neighborhood-metabolic-annotation

## Summary

A re-ranking skill that quantifies each STRING gene's percentage of first-degree neighbors annotated in HMDB and converts it into a per-gene weight p, producing the combined score C = p*Exomiser + (1-p)*metPropagate that corrects metPropagate's bias toward genes embedded in metabolic neighborhoods.

## When to use

Use when per-patient metPropagate LPA propagated scores and Exomiser-Phenotype candidate scores (Exomiser-filtered lists of ~281-609 genes) must be merged to boost causative-gene ranks for suspected IEM patients.

## When NOT to use

- No protein functional linkage network or gene-metabolite annotations are available (e.g., non-human organism or no STRING/HMDB coverage), so the per-gene p weight cannot be computed
- Input is raw LC/MS feature data requiring the XCMS/metabolomic-processing pipeline rather than already-prioritized candidate gene lists from Exomiser filtering
- Candidate lists are already very short (e.g., trio WES with parental data), so the top-20th-percentile binning and reweighting add little value

## Inputs

- Per-patient metPropagate LPA propagated scores (post-propagation 'Score.ID' files from integration/results/)
- Exomiser-Phenotype candidate gene scores per TIDEX patient
- Exomiser variant-filtered candidate gene lists (281-609 genes per patient, from DeepVariant singleton VCFs)
- STRING v11 functional network file (STRING_graph_file_v11_gene_list_functional_entire_db) and node_id-to-gene-name mapping file
- HMDB gene-metabolite annotations (HMDB_db_ver4.csv)

## Outputs

- Per-gene combined scores C = p*Exomiser + (1-p)*metPropagate and per-patient re-ranked candidate lists
- Per-gene HMDB first-degree-neighbor annotation percentages and ten 10%-interval bin assignments for 19,354 STRING genes
- combined_rankings.csv with per-gene scores, ranks, and per-patient summary counts

## How to apply

Load each patient's metPropagate LPA output (post-propagation node scores, e.g. integration/results/ 'Score.ID' files) and the Exomiser-Phenotype scores for the 11 TIDEX patients, scaling both vectors between 0 and 1. Using STRING v11 (STRING_graph_file_v11_gene_list_functional_entire_db plus the node_id-to-gene mapping) and HMDB gene-metabolite annotations (HMDB_db_ver4.csv), compute for each gene the percentage of first-degree neighbors annotated in HMDB, then bin all 19,354 STRING genes into ten bins of 10% intervals (0-10%, 10-20%, ...). Within each bin, compute p as the percentage of genes ranked in the top 20th percentile of all 19,354 genes; p then acts as the weight favoring Exomiser for genes in metabolic-dense neighborhoods and metPropagate elsewhere. For every candidate gene of every patient compute C = p*Exomiser + (1-p)*metPropagate, rescaling each term between 0 and 1 before summing, rank each patient's candidate genes by C, and check whether at least one causative gene falls in the top 20th percentile (rank <= 60 for 300-gene lists). Write combined per-gene scores, ranks, and per-patient summary counts to combined_rankings.csv.

## Related tools

- **metPropagate** (Supplies the propagated per-gene metabolomic scores (LPA output) that form the (1-p) weighted term of the combination) — github.com/emmagraham/metPropagate
- **Exomiser** (Supplies the phenotype-based candidate gene scores and variant-filtered candidate lists that form the p weighted term)
- **STRING** (Provides the v11 protein functional linkage network used to compute first-degree-neighbor HMDB annotation percentages)
- **yamaguchiyuto/label_propagation** (Label-propagation implementation adapted by metPropagate to propagate metabolomic enrichment scores across the STRING network) — https://github.com/yamaguchiyuto/label_propagation

## Examples

```
bash METABOLOMIC_PROCESSING_PIPELINE/run_met_processing.sh && python main.py hmn -g STRING_graph_file_v11_functional_entire_db.txt -l intermediate_label_file_sampleA -o integration/results/sampleA
```

## Evaluation signals

- The combined ranking places at least one causative gene in the top 20th percentile in 10/11 TIDEX patients
- The combined ranking improves the causative gene's rank relative to Exomiser-alone or metPropagate-alone in 8/11 patients
- All 19,354 STRING genes are partitioned into ten bins with 10% neighbor-annotation intervals, and p is a percentage (0-100, or 0-1 when normalized) within each bin
- Combined scores C lie in [0,1] because both terms are rescaled between 0 and 1 before summing with weights p and (1-p)
- Top 20th percentile corresponds to rank <= 60 for 300-gene lists (55-115 genes for the reported 281-609-gene candidate lists)

## Limitations

- metPropagate is biased toward genes whose network neighborhoods are dense in HMDB-annotated (metabolic) genes, which is the reason the p-weighted combination was needed
- Metabolites commonly used as IEM biomarkers, such as acyl glycines or acyl carnitines, are not annotated to IEM-associated genes in HMDB, weakening the metabolic signal for some diseases
- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities could not be determined
- The metabolomic system used could not measure large glycosaminoglycans, limiting analysis for the IDS patient
- No software version reported for Exomiser/hiPHIVE in the article; STRING version (v11) is specified only in the repository graph files

## Evidence

- [abstract] metPropagate uses UM data from a single patient and a group of controls to prioritize candidate genes: "Here, we present a network-based bioinformatics approach, metPropagate, that uses UM data from a single patient and a group of controls to prioritize candidate genes."
- [intro] STRING network scoring and propagation: "We then use this information to assign a score to each protein in a protein functional linkage network (STRING), and then propagate this evidence across the network"
- [results] Scaled DME scores as STRING weights: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [results] Candidate gene list sizes from Exomiser filters: "between 281 and 609 total genes emerged from Exomiser’s variant effect and population frequency filters"
- [readme] LPA output file format: "Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID""
