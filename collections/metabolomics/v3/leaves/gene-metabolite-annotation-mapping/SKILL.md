---
name: gene-metabolite-annotation-mapping
description: 'Use when an XCMS-style feature intensity matrix must be tied to gene
  function: exact-mass matching of m/z features to HMDB metabolites, removing unannotated
  features, and testing DAM enrichment among gene-associated metabolite sets.'
when_to_use_negative:
- No feature-to-metabolite mapping is available for the metabolomic system (as for
  the TIDEX dataset), so exact metabolite identities cannot be assigned
- Input is already a per-gene score vector (e.g., a finished ME file) — go directly
  to label propagation instead of re-mapping features
- Disease-relevant metabolites (e.g., acyl glycines, acyl carnitines used as IEM biomarkers)
  are not annotated to disease genes in HMDB, so enrichment mapping cannot connect
  them
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Produces the raw LC/MS feature intensity matrices and peak tables (patient
    vs. biofluid-specific controls) that the annotation mapping consumes
- name: metPropagate
  role: Contains the METABOLOMIC_PROCESSING_PIPELINE (preprocessing.R) implementing
    median-scaling, imputation, baseline normalization, HMDB annotation removal, exact-mass
    matching, and Fisher's Exact Test DAM enrichment
  repo: https://github.com/emmagraham/metPropagate
- name: HMDB (Human Metabolome Database)
  role: Source of gene-metabolite associations and known metabolite annotations used
    for feature filtering and per-gene metabolite sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:26d34f4b7fe5eed99164005d6fc49d9a9bfc0fabea8dcbe38a72e94149f20cb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmagraham/metPropagate
  tools:
  - HMDB (Human Metabolome Database)
  - XCMS
  - metPropagate
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gene-metabolite-annotation-mapping

## Summary

Links metabolomic features and metabolites to genes using HMDB gene-metabolite associations, so differentially abundant metabolites (DAMs) can be converted into per-gene metabolic enrichment (ME/DME) scores that seed downstream network propagation.

## When to use

Use when an XCMS-style feature intensity matrix must be tied to gene function: exact-mass matching of m/z features to HMDB metabolites, removing unannotated features, and testing DAM enrichment among gene-associated metabolite sets.

## When NOT to use

- No feature-to-metabolite mapping is available for the metabolomic system (as for the TIDEX dataset), so exact metabolite identities cannot be assigned
- Input is already a per-gene score vector (e.g., a finished ME file) — go directly to label propagation instead of re-mapping features
- Disease-relevant metabolites (e.g., acyl glycines, acyl carnitines used as IEM biomarkers) are not annotated to disease genes in HMDB, so enrichment mapping cannot connect them

## Inputs

- XCMS output intensity matrices for negative and positive ESI modes (e.g., metabolomics/negative_input_intensity_matrix.csv, positive_input_intensity_matrix.csv) and peak tables (negativepeakTable.csv, positivepeakTable.csv)
- HMDB gene-metabolite association database (HMDB_db_ver4.csv, zipped in the repo)
- Per-metabolite z-scores for the single patient versus biofluid-specific controls

## Outputs

- neg_mode/ or pos_mode/number_of_metabolites_matched_to_features.csv (metabolite IDs matched per feature after exact mass matching of m/z ratios)
- neg_mode/ or pos_mode/negative(or positive)_linear_mode_msea_genebig_allpeaks.csv (enrichment of DAMs for all genes in HMDB)
- output/linear_msea_genebig_allpeaks_combined.csv (combined positive- and negative-mode per-gene DAM enrichment)
- Per-gene ME/DME scores in [0,1] used as labels for label propagation

## How to apply

Process raw LC/MS intensities per ESI mode by median-scaling, missing value imputation with the minimum detected intensity value, linear baseline normalization, and a filter removing any feature not found in at least 10% of samples. Match each feature's input m/z ratio to possible metabolite IDs by exact mass and record the match count (number_of_metabolites_matched_to_features.csv), then remove features not annotated to any known metabolite in the HMDB and features annotated as non-base isotopes. Define DAMs as metabolites with absolute z-score >= 2 versus the biofluid-specific controls, and test statistical enrichment of DAMs among each gene's HMDB-associated metabolite group with a Fisher's Exact Test. Compute the per-gene ME score by multiplying the scaled enrichment p-value (differentially metabolic enrichment, DME) by the absolute value of each gene's largest metabolite z-score; DME scores are scaled between 0 and 1 to serve as continuous labels for the STRING label-propagation step.

## Related tools

- **XCMS** (Produces the raw LC/MS feature intensity matrices and peak tables (patient vs. biofluid-specific controls) that the annotation mapping consumes)
- **metPropagate** (Contains the METABOLOMIC_PROCESSING_PIPELINE (preprocessing.R) implementing median-scaling, imputation, baseline normalization, HMDB annotation removal, exact-mass matching, and Fisher's Exact Test DAM enrichment) — https://github.com/emmagraham/metPropagate
- **HMDB (Human Metabolome Database)** (Source of gene-metabolite associations and known metabolite annotations used for feature filtering and per-gene metabolite sets)

## Examples

```
bash METABOLOMIC_PROCESSING_PIPELINE/run_met_processing.sh
```

## Evaluation signals

- number_of_metabolites_matched_to_features.csv exists and gives a plausible count of possible metabolite IDs per feature after exact mass matching
- Features not annotated to any known HMDB metabolite and non-base isotope features are absent from downstream enrichment inputs
- The msea_genebig_allpeaks files exist per ESI mode and the combined linear_msea_genebig_allpeaks.csv covers both positive- and negative-mode DAMs
- Every gene in the enrichment output has an ME score scaled into [0,1] (DME scale) suitable as a continuous label
- DAM calls are reproducible at the stated cutoff: absolute z-score >= 2, with enrichment tested by Fisher's Exact Test

## Limitations

- Metabolites commonly used as IEM biomarkers, such as acyl glycines or acyl carnitines, are not annotated to IEM-associated genes in HMDB
- Some metabolites cannot be measured by a given chromatographic/MS combination (e.g., large glycosaminoglycans for the IDS patient), leaving the analysis insensitive to those diseases
- For TIDEX, feature-to-metabolite mapping was unavailable, so exact metabolite identities could not be determined

## Evidence

- [readme] Exact-mass matching of m/z features to HMDB metabolite IDs: "number_of_metabolites_matched_to_features.csv (number of possible metabolite IDs of each feature after exact mass matching of input m/z ratios)"
- [methods] HMDB-annotation and isotope filtering of features: "removal of (1) features not annotated to any known metabolites in the HMDB and (2) features annotated as non-base isotopes31"
- [results] DAM identification threshold: "Differentially abundant metabolites (DAMs) were identified as those with an absolute z-score greater than or equal to 2"
- [results] Fisher's Exact Test for per-gene DAM enrichment: "statistical enrichment of DAMs among each group of gene-associated metabolites was assessed using a Fisher’s Exact Test"
- [results] ME/DME score computation: "we multiped the scaled enrichment p-value (differentially metabolic enrichment—DME) by the absolute value of each gene’s largest metabolite z-score"
- [results] HMDB annotations enable gene prioritization from metabolomics: "metabolomic data and gene-metabolite associations available in the Human Metabolome Database (HMDB) could be used to prioritize at least one gene"
- [results] Downstream use: scaled DME weights seed the STRING network: "DME scores were scaled between 0 and 1 and used as weights assigned to each gene in the STRING network"
