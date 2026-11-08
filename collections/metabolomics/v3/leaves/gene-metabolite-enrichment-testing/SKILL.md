---
name: gene-metabolite-enrichment-testing
description: Use when you have a patient-vs-control metabolite z-score matrix and
  need to convert DAMs into per-gene enrichment scores via gene-metabolite associations
  before ranking candidate genes.
when_to_use_negative:
- The metabolomic system cannot resolve feature-to-metabolite mapping (as for the
  TIDEX cohort), so DAMs cannot be assigned HMDB identities for enrichment testing
- The disease-relevant metabolites are unmeasurable or unannotated in HMDB (e.g.,
  large glycosaminoglycans for the IDS patient, or acyl glycines/acyl carnitines not
  annotated to IEM genes), making the gene-level enrichment insensitive
- You only need variant-level prioritization without metabolomic evidence — Exomiser's
  phenotype-driven ranking alone is the appropriate comparator in that setting
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_1772
tools:
- name: metPropagate
  role: 'Implements the DAM enrichment pipeline: Fisher''s Exact Test of patient DAMs
    against each gene''s HMDB metabolite set and computation of scaled ME scores'
  repo: github.com/emmagraham/metPropagate
- name: HMDB
  role: Source of gene-metabolite associations defining each gene's metabolite set
    for the overrepresentation test (HMDB_db_ver4.csv)
- name: tidyverse
  role: R packages (tidyverse, CAMERA, stringr, reshape2) used in METABOLOMIC_PROCESSING_PIPELINE
    to process XCMS output and compute per-gene enrichments
  repo: github.com/emmagraham/metPropagate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:27bb2cb669419434e412b1b88af4dcdf70fd860853fc059f1bd7d76bea7605cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - NumPy
  - metPropagate
  - scikit-learn
  - tidyverse
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_1772
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-metabolite-enrichment-testing

## Summary

Computes per-gene metabolic enrichment (ME/DME) scores by testing whether a patient's differentially abundant metabolites (DAMs) are over-represented among each gene's HMDB-curated metabolite set, then scales the scores 0-1 for use as network labels. This is the pre-propagation step of metPropagate that converts a metabolite z-score matrix into per-gene labels.

## When to use

Use when you have a patient-vs-control metabolite z-score matrix and need to convert DAMs into per-gene enrichment scores via gene-metabolite associations before ranking candidate genes.

## When NOT to use

- The metabolomic system cannot resolve feature-to-metabolite mapping (as for the TIDEX cohort), so DAMs cannot be assigned HMDB identities for enrichment testing
- The disease-relevant metabolites are unmeasurable or unannotated in HMDB (e.g., large glycosaminoglycans for the IDS patient, or acyl glycines/acyl carnitines not annotated to IEM genes), making the gene-level enrichment insensitive
- You only need variant-level prioritization without metabolomic evidence — Exomiser's phenotype-driven ranking alone is the appropriate comparator in that setting

## Inputs

- Metabolite z-score matrix from a single patient vs. biofluid-specific controls (e.g., 436 plasma metabolite z-scores for one Miller patient)
- Per-feature metabolite IDs or exact-mass matches to HMDB entries (from XCMS peak tables: negativepeakTable.csv / positivepeakTable.csv)
- HMDB gene-metabolite association database (HMDB_db_ver4.csv; hmdb.ca parse)
- XCMS intensity matrices per ESI mode (negative/positive_input_intensity_matrix.csv)

## Outputs

- Per-gene DAM enrichment table for all genes in HMDB (e.g., neg_mode/positive_linear_mode_msea_genebig_allpeaks.csv)
- Combined enrichment file linear_msea_genebig_allpeaks_combined.csv merging positive- and negative-mode DAM enrichments
- number_of_metabolites_matched_to_features.csv (count of possible metabolite IDs per feature after exact mass matching of m/z ratios)
- Scaled 0-1 per-gene ME/DME scores used as label-propagation labels (label_files such as intermediate_label_file_sampleA)

## How to apply

Start from a metabolite z-score matrix (e.g., 436 plasma metabolite z-scores referenced to non-IEM controls) and define DAMs as features with absolute z-score >= 2. For every candidate gene, test overrepresentation of the patient's DAMs among the gene's HMDB-curated metabolite set (parsed from hmdb.ca) with Fisher's Exact Test; in the Miller benchmark this used HMDB annotations from April 1st 2019. Compute the ME score as the scaled enrichment p-value (DME) multiplied by the absolute value of the gene's largest metabolite z-score, so genes annotated to strongly perturbed metabolites score highest. Scale each ME score between 0 and 1 before writing the per-gene enrichment file; in the repo pipeline this produces per-mode files (e.g., negative/positive linear_mode_msea_genebig_allpeaks.csv) that are combined into linear_msea_genebig_allpeaks_combined.csv for both ESI+ and ESI- mode DAMs. Downstream, these scaled scores become label values (STRING_sampleA / intermediate_label_file_sampleA) fed to label propagation over the STRING network.

## Related tools

- **metPropagate** (Implements the DAM enrichment pipeline: Fisher's Exact Test of patient DAMs against each gene's HMDB metabolite set and computation of scaled ME scores) — github.com/emmagraham/metPropagate
- **HMDB** (Source of gene-metabolite associations defining each gene's metabolite set for the overrepresentation test (HMDB_db_ver4.csv))
- **tidyverse** (R packages (tidyverse, CAMERA, stringr, reshape2) used in METABOLOMIC_PROCESSING_PIPELINE to process XCMS output and compute per-gene enrichments) — github.com/emmagraham/metPropagate

## Examples

```
bash METABOLOMIC_PROCESSING_PIPELINE/run_met_processing.sh
```

## Evaluation signals

- DAM count is reproducible: features with |z-score| >= 2 per patient should match the reported DAM definition before any enrichment is run
- Every gene's enrichment is computed against its HMDB-annotated metabolite set only; features with no HMDB annotation must have been removed upstream
- All ME scores must fall in [0, 1] after scaling, since they are used directly as label values for propagation
- Combined enrichment file must merge both ESI modes: linear_msea_genebig_allpeaks_combined.csv should reflect positive- and negative-mode DAMs together
- Downstream check: after label propagation over STRING v11 (19,354 genes, lambda=0.99), at least one causative gene should rank in the top 20th percentile (rank <= 60 of 300) for ~92% (98/107) of Miller patients

## Limitations

- Metabolites commonly used as IEM biomarkers (e.g., acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB, weakening enrichment for those diseases
- If the metabolomic platform misses the disease metabolites entirely (e.g., large glycosaminoglycans unmeasurable for the IDS patient), the enrichment signal is absent
- Feature-to-metabolite mapping may be unavailable for a given system, so exact metabolite identities — and hence correct gene annotation — cannot be determined

## Evidence

- [results] DAMs identified as z-score >= 2: "Differentially abundant metabolites (DAMs) were identified as those with an absolute z-score greater than or equal to 2"
- [results] Fisher's Exact Test for DAM enrichment: "statistical enrichment of DAMs among each group of gene-associated metabolites was assessed using a Fisher’s Exact Test"
- [results] ME score definition: "we multiped the scaled enrichment p-value (differentially metabolic enrichment—DME) by the absolute value of each gene’s largest metabolite z-score"
- [results] HMDB as the association source: "metabolomic data and gene-metabolite associations available in the Human Metabolome Database (HMDB) could be used to prioritize at least one gene"
- [readme] Combined per-gene enrichment output: "output/linear_msea_genebig_allpeaks_combined.csv (enrichment of DAMs for all genes in HMDB, when combining both positive and negative mode DAMs)"
- [methods] HMDB feature filtering prerequisite: "removal of (1) features not annotated to any known metabolites in the HMDB and (2) features annotated as non-base isotopes31"
