---
name: metabolomic-enrichment-score-scaling
description: Use when you have per-patient DAM lists (absolute z-score >= 2) from
  XCMS untargeted LC-MS output and need continuous [0,1] per-gene seed labels for
  metPropagate label propagation over STRING.
when_to_use_negative:
- Candidate gene ranking from propagated scores is already done — this skill only
  produces pre-propagation seed labels, not final Score.ID rankings.
- No differentially abundant metabolites exist for the patient (e.g., disease-associated
  metabolites like large glycosaminoglycans are not measurable by the metabolomic
  system), so no meaningful ME seed signal can be computed.
- Input is raw LC-MS feature intensities that have not yet passed XCMS peak detection,
  10% sample-presence filtering, HMDB annotation, and non-base-isotope removal.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: metPropagate
  role: Pipeline whose METABOLOMIC_PROCESSING_PIPELINE computes Fisher's Exact DAM
    enrichment against HMDB gene sets and produces the combined enrichment and label
    files used for score scaling
  repo: https://github.com/emmagraham/metPropagate
- name: HMDB
  role: Source of gene-metabolite association sets over which DAM enrichment is tested
    and from which unannotated features are removed
- name: XCMS
  role: Generates the raw intensity matrices and peak tables from which metabolite
    z-scores and DAMs (|z| >= 2) are derived
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:026ea8684364bfeeedb5d1b3ccda7cb150f43b9e97e6e5263a8028f7d3527d23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - Exomiser
  - HMDB
  - XCMS
  - metPropagate
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metabolomic-enrichment-score-scaling

## Summary

Computes per-gene metabolic enrichment (ME) scores from patient-specific untargeted LC-MS data — Fisher's Exact enrichment of DAMs against HMDB gene-metabolite sets, multiplied by the gene's largest metabolite z-score — and scales them to [0,1] for use as seed labels in network label propagation.

## When to use

Use when you have per-patient DAM lists (absolute z-score >= 2) from XCMS untargeted LC-MS output and need continuous [0,1] per-gene seed labels for metPropagate label propagation over STRING.

## When NOT to use

- Candidate gene ranking from propagated scores is already done — this skill only produces pre-propagation seed labels, not final Score.ID rankings.
- No differentially abundant metabolites exist for the patient (e.g., disease-associated metabolites like large glycosaminoglycans are not measurable by the metabolomic system), so no meaningful ME seed signal can be computed.
- Input is raw LC-MS feature intensities that have not yet passed XCMS peak detection, 10% sample-presence filtering, HMDB annotation, and non-base-isotope removal.

## Inputs

- XCMS output: positive/negative_input_intensity_matrix.csv and peakTable.csv (patient vs. biofluid-specific controls)
- Per-gene metabolomic enrichment file linear_msea_genebig_allpeaks_combined.csv (Fisher's Exact enrichment of DAMs against HMDB gene sets)
- HMDB gene-metabolite association database (HMDB_db_ver4.csv)
- Metabolite z-scores per patient (e.g., 436 plasma metabolite z-scores in the Miller data set)

## Outputs

- Per-gene ME score table (scaled enrichment p-value x max |z-score| per gene)
- Scaled [0,1] label vector y per gene for label propagation (intermediate_label_file / STRING_sampleA label files)
- Combined positive+negative mode enrichment file linear_msea_genebig_allpeaks_combined.csv

## How to apply

First define differentially abundant metabolites (DAMs) as features with absolute z-score >= 2 relative to biofluid-specific controls. For each gene, assess statistical enrichment of DAMs among its HMDB gene-associated metabolites using a Fisher's Exact Test (the linear_msea_genebig_allpeaks_combined.csv enrichment, combining positive and negative ESI modes). Compute the ME score by multiplying the scaled enrichment p-value (the DME score) by the absolute value of the gene's largest metabolite z-score, so that genes whose most dysregulated metabolite is extreme receive higher seed signal. Finally scale DME/ME scores to continuous labels y in [0,1] before assigning them as weights to genes in the STRING v11 network for label propagation with lambda = 0.99.

## Related tools

- **metPropagate** (Pipeline whose METABOLOMIC_PROCESSING_PIPELINE computes Fisher's Exact DAM enrichment against HMDB gene sets and produces the combined enrichment and label files used for score scaling) — https://github.com/emmagraham/metPropagate
- **HMDB** (Source of gene-metabolite association sets over which DAM enrichment is tested and from which unannotated features are removed)
- **XCMS** (Generates the raw intensity matrices and peak tables from which metabolite z-scores and DAMs (|z| >= 2) are derived)

## Examples

```
bash METABOLOMIC_PROCESSING_PIPELINE/run_met_processing.sh
```

## Evaluation signals

- Every computed label y lies within [0,1] after scaling of the DME/ME scores.
- DAM call set matches the |z-score| >= 2 threshold applied to the patient's metabolite z-scores.
- Enrichment output covers all genes in HMDB for both ESI modes and is combined into linear_msea_genebig_allpeaks_combined.csv.
- Genes with no DAMs among their associated metabolites receive an ME score of 0 (or no score), consistent with the ablation arm where causative genes lacked an ME score.
- The combined enrichment file, moved to integration/enrichment_files as <sample>_enrichment_file.csv, yields a label file acceptable to the label propagation script (run_label_prop_cluster.sh).

## Limitations

- Feature-to-metabolite mapping was unavailable for the TIDEX metabolomic system, so exact metabolite identities could not be determined and enrichment relies on feature-level HMDB matching.
- Common IEM biomarker metabolites such as acyl glycines or acyl carnitines are not annotated to IEM-associated genes in HMDB, weakening ME signal for some diseases.
- Some metabolites are unmeasurable by a given metabolomic system (e.g., large glycosaminoglycans for the IDS patient), leaving the analysis insensitive to diseases associated with them.

## Evidence

- [results] DAM threshold: "Differentially abundant metabolites (DAMs) were identified as those with an absolute z-score greater than or equal to 2"
- [results] ME score formula: [evidence span withheld — anchor status: not_found]
- [results] Fisher enrichment: "statistical enrichment of DAMs among each group of gene-associated metabolites was assessed using a Fisher’s Exact Test"
- [results] Scaling for STRING: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
- [readme] Combined enrichment output: "output/linear_msea_genebig_allpeaks_combined.csv (enrichment of DAMs for all genes in HMDB, when combining both positive and negative mode DAMs)"
