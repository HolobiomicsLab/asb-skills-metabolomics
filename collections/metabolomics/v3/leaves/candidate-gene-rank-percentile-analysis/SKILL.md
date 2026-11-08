---
name: candidate-gene-rank-percentile-analysis
description: Use when you have per-gene propagated network scores and a WES-derived
  candidate gene list (e.g., 281-609 Exomiser genes) and need to assess whether causative
  genes rank in the top 20th percentile.
when_to_use_negative:
- Input is raw untargeted LC-MS feature intensity data — this skill starts from already-propagated
  gene scores, not XCMS output
- No known causative gene exists for the patient, making percentile-hit evaluation
  undefined (prioritization itself may still be exploratory)
- Candidate gene lists are far shorter than 281 genes (e.g., trio WES-filtered lists),
  where the top-20th-percentile metric calibrated on 281-609-gene lists becomes less
  comparable
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3173
tools:
- name: metPropagate
  role: Produces the propagated per-gene scores (Score.ID) from metabolomic enrichment
    and the STRING network that are ranked in this skill
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Generates the per-patient candidate gene lists (281-609 genes) via variant
    effect and population frequency filtering; its hiPHIVE ranking serves as the comparison
    baseline
- name: STRING
  role: Functional linkage network (v11) over which ME scores were propagated before
    candidate ranking
- name: yamaguchiyuto/label_propagation
  role: Python implementation of the LGC label-propagation solver (alpha = 0.99) used
    to compute the propagated scores
  repo: https://github.com/yamaguchiyuto/label_propagation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:d212cb3443b47f6fd3c491093407f8df03c77126741a104360ee1f7114d21aa9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - STRING
  - metPropagate
  - yamaguchiyuto/label_propagation
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# candidate-gene-rank-percentile-analysis

## Summary

Ranks a patient's Exomiser candidate gene list by propagated metPropagate score (Score.ID) and evaluates whether at least one known causative gene falls in the top 20th percentile, the success metric used across the 11 TIDEX neurometabolic patients.

## When to use

Use when you have per-gene propagated network scores and a WES-derived candidate gene list (e.g., 281-609 Exomiser genes) and need to assess whether causative genes rank in the top 20th percentile.

## When NOT to use

- Input is raw untargeted LC-MS feature intensity data — this skill starts from already-propagated gene scores, not XCMS output
- No known causative gene exists for the patient, making percentile-hit evaluation undefined (prioritization itself may still be exploratory)
- Candidate gene lists are far shorter than 281 genes (e.g., trio WES-filtered lists), where the top-20th-percentile metric calibrated on 281-609-gene lists becomes less comparable

## Inputs

- Per-patient metabolomic enrichment file (linear_msea_genebig_allpeaks_combined.csv) converted to ME/DME gene scores and scaled labels y in [0,1]
- STRING v11 functional network file (STRING_graph_file_v11_functional_entire_db) and gene-to-nodeid mapping file
- Exomiser candidate gene list per patient (sampleA_candidate_genes.csv; 281-609 genes per TIDEX patient)
- Known causative gene(s) per patient for percentile evaluation

## Outputs

- Per-patient propagated-score file listing Score.ID (propagated score) and Original.Score.ID for each candidate gene
- Summary table of causative-gene ranks and top-20th-percentile hit/miss across all 11 patients

## How to apply

After label propagation over the STRING v11 functional network (Zhou et al. LGC formulation with alpha/lambda = 0.99), extract each node's propagated Score.ID alongside Original.Score.ID from the LPA_output file and subset to the patient's candidate gene list. Rank the candidate genes in descending order of propagated score, then compute the top-20th-percentile cutoff (rank <= ~60 for a 300-gene list; 55-115 genes for the 281-609-gene TIDEX lists). Check whether at least one known causative gene falls within this percentile; record the per-patient hit/miss. Repeat across all patients and summarize (target: >= 9/11 TIDEX patients, i.e., 9/13 causative genes, using uncurated LC-MS data). Optionally compare against Exomiser's phenotype-based (hiPHIVE) ranking and a weighted combination of both scores, which improved prioritization.

## Related tools

- **metPropagate** (Produces the propagated per-gene scores (Score.ID) from metabolomic enrichment and the STRING network that are ranked in this skill) — github.com/emmagraham/metPropagate
- **Exomiser** (Generates the per-patient candidate gene lists (281-609 genes) via variant effect and population frequency filtering; its hiPHIVE ranking serves as the comparison baseline)
- **STRING** (Functional linkage network (v11) over which ME scores were propagated before candidate ranking)
- **yamaguchiyuto/label_propagation** (Python implementation of the LGC label-propagation solver (alpha = 0.99) used to compute the propagated scores) — https://github.com/yamaguchiyuto/label_propagation

## Examples

```
python main.py lgc -g STRING_graph_file_v11_functional_entire_db -l intermediate_label_file_sampleA -o sampleA.output  # then rank candidate genes by Score.ID and check top-20th-percentile hit
```

## Evaluation signals

- At least one causative gene should fall within the top 20th percentile in >= 9/11 TIDEX patients (9/13 causative genes), matching the reported result
- Output files must contain both Score.ID and Original.Score.ID columns for every candidate gene, as produced by run_label_prop_cluster.sh
- Top-20th-percentile cutoff must equal 55-115 genes for the 281-609-gene TIDEX candidate lists (rank ~60 for a 300-gene list)
- Propagated Score.ID values should remain within the original label range [0,1] and differ from Original.Score.ID where network neighbors smoothed the signal
- Per-patient ranking of metPropagate scores can be benchmarked against Exomiser's hiPHIVE ranking (causative gene ranked higher in 6/11 TIDEX patients in the paper)

## Limitations

- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities could not be determined, relying on uncurated HMDB annotations
- The metabolic system could not measure large glycosaminoglycans, limiting ME analysis for the IDS patient
- Metabolites commonly used as IEM biomarkers (e.g., acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB
- Analyses used singleton exomes rather than trio WES, which in practice would shorten candidate lists and simplify prioritization

## Evidence

- [abstract] metPropagate placed at least one causative gene in the top 20th percentile in 9/11 patients: "metPropagate placed at least one causative gene in the top 20th percentile in 9/11 patients"
- [discussion] candidate gene lists include 281-609 genes: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [results] Exomiser filtering produced 281-609 genes per patient: "between 281 and 609 total genes emerged from Exomiser’s variant effect and population frequency filters"
- [readme] Results populate with Score.ID and Original.Score.ID: "Results of label propagation will populate in *integration/results/*. File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as"
- [results] DME scores scaled 0-1 and used as STRING weights: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
