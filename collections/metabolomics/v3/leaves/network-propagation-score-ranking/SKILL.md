---
name: network-propagation-score-ranking
description: Use when a single patient's untargeted metabolomics yields DAMs (|z|
  >= 2) plus an Exomiser-filtered candidate gene list, and genes must be ranked by
  metabolomics-derived scores propagated across a STRING protein interaction network.
when_to_use_negative:
- Input is still raw LC/MS feature tables (negative/positive intensity matrices, peakTables)
  — these must first pass through METABOLOMIC_PROCESSING_PIPELINE (median-scaling,
  min-intensity imputation, HMDB annotation) to produce per-gene enrichment scores.
- The disease's diagnostic metabolites are outside the platform's measurable space
  or HMDB annotation (e.g. large glycosaminoglycans for the IDS patient; acyl glycines/acyl
  carnitines not annotated to IEM genes) — propagation cannot recover absent signal.
- No patient metabolomics exists and only HPO phenotype terms are available — Exomiser's
  hiPHIVE phenotype-driven ranking is the appropriate tool instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: metPropagate
  role: 'End-to-end implementation: computes DME/ME enrichment scores, runs the label
    propagation step over STRING, and writes Score.ID/Original.Score.ID ranking files
    to LPA_output'
  repo: https://github.com/emmagraham/metPropagate
- name: STRING
  role: Supplies the v11 protein functional linkage network used as the propagation
    graph (STRING_graph_file_v11_gene_list_functional_entire_db with node_id-to-gene
    mapping), downloaded from stringdb.org
- name: label_propagation
  role: Scikit-learn-compatible label propagation implementations (HMN, LGC, PARW,
    OMNIProp, CAMLP) that metPropagate's LPA component is adapted from
  repo: https://github.com/yamaguchiyuto/label_propagation
- name: Exomiser
  role: Produces the per-patient candidate gene lists (variant-effect and population-frequency
    filtered, e.g. sampleA_candidate_genes.csv) that are ranked by propagated scores;
    its hiPHIVE phenotype score is the comparison ranking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:ff48ced350bd961c185520677bbb8d5e11ae0e173a42a0ef860c012fe00538f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - Exomiser
  - STRING
  - label_propagation
  - metPropagate
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# network-propagation-score-ranking

## Summary

metPropagate's core ranking skill: per-gene metabolic enrichment (ME/DME) scores derived from a patient's untargeted metabolomics are label-propagated across the STRING v11 protein functional linkage network, and Exomiser-filtered candidate genes are ranked by the post-propagation score (Score.ID) to prioritize metabolic disease genes.

## When to use

Use when a single patient's untargeted metabolomics yields DAMs (|z| >= 2) plus an Exomiser-filtered candidate gene list, and genes must be ranked by metabolomics-derived scores propagated across a STRING protein interaction network.

## When NOT to use

- Input is still raw LC/MS feature tables (negative/positive intensity matrices, peakTables) — these must first pass through METABOLOMIC_PROCESSING_PIPELINE (median-scaling, min-intensity imputation, HMDB annotation) to produce per-gene enrichment scores.
- The disease's diagnostic metabolites are outside the platform's measurable space or HMDB annotation (e.g. large glycosaminoglycans for the IDS patient; acyl glycines/acyl carnitines not annotated to IEM genes) — propagation cannot recover absent signal.
- No patient metabolomics exists and only HPO phenotype terms are available — Exomiser's hiPHIVE phenotype-driven ranking is the appropriate tool instead.

## Inputs

- Per-sample metabolomic enrichment file from the processing pipeline (e.g. linear_msea_genebig_allpeaks_combined.csv, renamed <samplename>_enrichment_file.csv)
- STRING v11 functional network edgelist (STRING_graph_file_v11_gene_list_functional_entire_db) plus node_id-to-gene-name mapping file
- Per-sample label file for label propagation (e.g. intermediate_label_file_sampleA)
- Per-patient Exomiser candidate gene list (sampleA_candidate_genes.csv; 281-609 genes after variant-effect and population-frequency filters)
- Known causative gene per patient (e.g. from Supplementary Table 1) when validating against a diagnosis

## Outputs

- LPA_output / integration/results file with per-gene post-propagation scores as Score.ID and original ME scores as Original.Score.ID
- Per-patient ranked candidate gene list sorted by descending propagated Score.ID
- Optional per-patient comparison table of metPropagate vs Exomiser-Phenotype causative-gene ranks (metpropagate_vs_exomiser_rank_comparison.csv)

## How to apply

Identify differentially abundant metabolites (DAMs) as features with absolute z-score >= 2 versus biofluid-specific controls, test DAM enrichment among HMDB gene-associated metabolites with Fisher's Exact Test, and compute each gene's DME score, scaled between 0 and 1 and multiplied by the absolute value of the gene's largest metabolite z-score to give the ME score. Assign these ME scores as weights/labels to genes in the STRING v11 functional linkage network (unzip STRING_graph_file_v11_functional_entire_db.txt.zip and use the node_id-to-gene mapping), then run label propagation with the code adapted from yamaguchiyuto/label_propagation: edit the sample_name variable in label_propagation/run_label_prop_cluster.sh and execute it, so results populate in integration/results/. Read the LPA_output file, which stores each candidate gene's post-propagation score as Score.ID and the pre-propagation score as Original.Score.ID, and sort the patient's candidate genes (Exomiser variant-effect/population-frequency filtered lists of 281-609 genes per TIDEX patient) in descending Score.ID order. For a diagnosed patient, record the causative gene's rank (top 20th percentile = 55-115 genes at these list sizes) and optionally compare it head-to-head with the Exomiser-Phenotype (hiPHIVE) rank; on TIDEX, metPropagate ranked the causative gene more highly in 6/11 patients.

## Related tools

- **metPropagate** (End-to-end implementation: computes DME/ME enrichment scores, runs the label propagation step over STRING, and writes Score.ID/Original.Score.ID ranking files to LPA_output) — https://github.com/emmagraham/metPropagate
- **STRING** (Supplies the v11 protein functional linkage network used as the propagation graph (STRING_graph_file_v11_gene_list_functional_entire_db with node_id-to-gene mapping), downloaded from stringdb.org)
- **label_propagation** (Scikit-learn-compatible label propagation implementations (HMN, LGC, PARW, OMNIProp, CAMLP) that metPropagate's LPA component is adapted from) — https://github.com/yamaguchiyuto/label_propagation
- **Exomiser** (Produces the per-patient candidate gene lists (variant-effect and population-frequency filtered, e.g. sampleA_candidate_genes.csv) that are ranked by propagated scores; its hiPHIVE phenotype score is the comparison ranking)

## Examples

```
bash label_propagation/run_label_prop_cluster.sh
```

## Evaluation signals

- Schema check: the output file in integration/results/ (LPA_output) contains per-candidate-gene columns Score.ID (post-propagation) and Original.Score.ID (original ME score).
- Ranking invariant: candidate genes are sorted in descending Score.ID, and Score.ID values differ from Original.Score.ID after propagation across the STRING network.
- Candidate list size invariant: after Exomiser variant-effect and population-frequency filtering, each TIDEX patient's list holds 281-609 genes, so the top 20th percentile equals 55-115 genes.
- Benchmark recall: at least one causative gene lands in the top 20th percentile for ~92% of Miller patients and 9/11 TIDEX patients.
- Head-to-head expectation on TIDEX: metPropagate ranks the causative gene more highly than the Exomiser-Phenotype (hiPHIVE) ranking in 6/11 patients (Exomiser-Phenotype higher in 5/11 patients and 7/13 genes).

## Limitations

- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities of features could not be determined.
- The platform could not measure large glycosaminoglycans, limiting ME analysis for the IDS patient; the analysis is insensitive to diseases associated with unmeasured metabolites.
- Common IEM biomarkers (acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB.
- Analyses used singleton exomes rather than trio WES; trio parental data would shorten the 281-609 gene candidate lists, so reported ranks are conservative, and no STRING/Exomiser software versions are reported.

## Evidence

- [abstract] ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients: "ranked the causative gene more highly than Exomiser’s phenotype-based ranking in 6/11 patients"
- [intro] assign a score to each protein in a protein functional linkage network (STRING), and then propagate this evidence across the network: "We then use this information to assign a score to each protein in a protein functional linkage network (STRING), and then propagate this evidence across the network"
- [intro] Each patient’s set of candidate genes is ranked by the resulting propagated score, resulting in a prioritized list of candidate genes: "Each patient’s set of candidate genes is ranked by the resulting propagated score, resulting in a prioritized list of candidate genes"
- [results] DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [results] Differentially abundant metabolites (DAMs) were identified as those with an absolute z-score greater than or equal to 2: "Differentially abundant metabolites (DAMs) were identified as those with an absolute z-score greater than or equal to 2"
- [readme] File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID".: "File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID"."
