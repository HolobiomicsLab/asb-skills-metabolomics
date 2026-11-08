---
name: metabolite-differential-abundance-zscore-analysis
description: Use when raw untargeted metabolomics peak tables (e.g. XCMS positive/negative
  mode intensity matrices comparing one patient to biofluid-specific controls) must
  be normalized, filtered, and converted into DAM calls and gene-level metabolic enrichment
  (ME) inputs.
when_to_use_negative:
- When no feature-to-metabolite mapping exists for the metabolomic system (as in the
  TIDEX dataset), exact metabolite identities cannot be determined and HMDB-based
  enrichment per gene is not meaningful
- When the disease's key biomarkers are metabolites not annotated to IEM-associated
  genes in HMDB (e.g. acyl glycines, acyl carnitines) or are unmeasurable (e.g. large
  glycosaminoglycans for the IDS patient)
- When the goal is phenotype/variant-based prioritization without metabolomics data
  — use a tool like Exomiser's hiPHIVE instead
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: metPropagate
  role: Hosts the METABOLOMIC_PROCESSING_PIPELINE (R preprocessing.R + run_met_processing.sh)
    that converts XCMS matrices into DAM enrichment files; downstream label propagation
    consumes its output
  repo: github.com/emmagraham/metPropagate
- name: XCMS
  role: Produces the raw LC/MS peak tables and patient-vs-control intensity matrices
    that are the pipeline's input
- name: HMDB
  role: Provides the gene-metabolite associations used for feature annotation filtering
    and Fisher's Exact Test enrichment of DAMs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:404da95aea30f61a0e078a7291a71aac029544c43af313712f6fd2e1876bb015
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - NumPy
  - XCMS
  - metPropagate
  - scikit-learn
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# metabolite-differential-abundance-zscore-analysis

## Summary

Processes untargeted metabolomics feature intensity matrices (XCMS ESI+/ESI- output) into per-patient differentially abundant metabolite (DAM) lists and per-gene metabolic enrichment scores, the preprocessing core that feeds metPropagate's label propagation.

## When to use

Use when raw untargeted metabolomics peak tables (e.g. XCMS positive/negative mode intensity matrices comparing one patient to biofluid-specific controls) must be normalized, filtered, and converted into DAM calls and gene-level metabolic enrichment (ME) inputs.

## When NOT to use

- When no feature-to-metabolite mapping exists for the metabolomic system (as in the TIDEX dataset), exact metabolite identities cannot be determined and HMDB-based enrichment per gene is not meaningful
- When the disease's key biomarkers are metabolites not annotated to IEM-associated genes in HMDB (e.g. acyl glycines, acyl carnitines) or are unmeasurable (e.g. large glycosaminoglycans for the IDS patient)
- When the goal is phenotype/variant-based prioritization without metabolomics data — use a tool like Exomiser's hiPHIVE instead

## Inputs

- XCMS positive-mode and negative-mode feature intensity matrices (e.g. positive_input_intensity_matrix.csv, negative_input_intensity_matrix.csv) for one patient vs controls
- XCMS peak tables (e.g. positivepeakTable.csv, negativepeakTable.csv) with m/z ratios for exact-mass feature-to-metabolite matching
- HMDB metabolite-gene association database (HMDB_db_ver4.csv shipped with the repo)
- Biofluid-specific control group intensities for z-score reference

## Outputs

- Per-mode scaled intensity matrices (linear_raw_intensities.csv, log_raw_intensities.csv, unnormalized_raw_intensities.csv)
- Per-gene DAM enrichment tables (negative/positive_linear_mode_msea_genebig_allpeaks.csv)
- Combined enrichment file linear_msea_genebig_allpeaks_combined.csv used as label-propagation input
- number_of_metabolites_matched_to_features.csv (feature-to-metabolite match counts after exact mass matching)
- significant_features_linear.csv (feature IDs of significant features across all samples)

## How to apply

Run the metPropagate METABOLOMIC_PROCESSING_PIPELINE separately on negative- and positive-mode XCMS intensity matrices: apply median-scaling, impute missing values with the minimum detected intensity, drop features absent from at least 10% of all samples, apply linear baseline normalization per feature, then remove features not annotated to any known HMDB metabolite and non-base isotopes. Compute patient z-scores referenced to the control group and call DAMs as features with absolute z-score >= 2. For each candidate gene, test enrichment of the patient's DAMs among the gene's HMDB-curated metabolite set with Fisher's Exact Test, and combine the per-mode gene-enrichment tables into a single file such as linear_msea_genebig_allpeaks_combined.csv, which becomes the label file for downstream label propagation on the STRING network.

## Related tools

- **metPropagate** (Hosts the METABOLOMIC_PROCESSING_PIPELINE (R preprocessing.R + run_met_processing.sh) that converts XCMS matrices into DAM enrichment files; downstream label propagation consumes its output) — github.com/emmagraham/metPropagate
- **XCMS** (Produces the raw LC/MS peak tables and patient-vs-control intensity matrices that are the pipeline's input)
- **HMDB** (Provides the gene-metabolite associations used for feature annotation filtering and Fisher's Exact Test enrichment of DAMs)

## Examples

```
bash METABOLOMIC_PROCESSING_PIPELINE/run_met_processing.sh
```

## Evaluation signals

- DAMs must be defined consistently: only features with |z-score| >= 2 relative to the control group should enter the enrichment step
- All three scaling outputs (linear, log, unnormalized) must be produced per ESI mode before enrichment
- The combined enrichment file linear_msea_genebig_allpeaks_combined.csv must exist and be moved to integration/enrichment_files renamed as <sample>_enrichment_file.csv
- number_of_metabolites_matched_to_features.csv should show plausible feature-to-metabolite match counts from exact-mass matching of input m/z ratios
- Enrichment p-values must come from Fisher's Exact Test per gene over its HMDB metabolite set, then be scaled between 0 and 1 before multiplication by the gene's largest-magnitude metabolite z-score

## Limitations

- Features not annotated to any known HMDB metabolite and non-base isotope features are discarded, shrinking the analyzable feature space
- Chromatographic/MS acquisition cannot measure all metabolites, so analyses are insensitive to diseases associated with unmeasured metabolites
- Common IEM biomarkers (acyl glycines, acyl carnitines) are not annotated to IEM-associated genes in HMDB, limiting direct ME signal
- The TIDEX metabolomic system lacked feature-to-metabolite mapping, so exact metabolite identities could not be determined

## Evidence

- [results] DAMs defined at |z|>=2: "Differentially abundant metabolites (DAMs) were identified as those with an absolute z-score greater than or equal to 2"
- [methods] Normalization and imputation: "Features underwent median-scaling and missing value imputation with the minimum detected intensity value"
- [methods] Presence filter: "filtering which removed any feature not found in at least 10% of all samples"
- [methods] HMDB annotation and isotope removal: "removal of (1) features not annotated to any known metabolites in the HMDB and (2) features annotated as non-base isotopes31"
- [results] Fisher enrichment step: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline input provenance: "The sample metabolomics files provided here are the output of an XCMS analysis comparing a single patient to a group of controls."
- [results] HMDB annotation gap for IEM biomarkers: "metabolites commonly used as IEM biomarkers, such as acyl glycines or acyl carnitines, are not annotated to IEM-associated genes in HMDB"
