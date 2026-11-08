---
name: wgcna-module-detection-metabolomics
description: Use when a normalized metabolite measurement table (e.g. Metabolon®-style
  data with SUB_PATHWAY annotations) should be partitioned into co-abundance modules
  and tested against sample traits such as tumor vs normal status or gender, without
  predefined metabolite sets.
when_to_use_negative:
- Input is raw mass-spectrometry spectra rather than a processed table of metabolite
  measurements — MetaboDiff explicitly 'starts off with the table of metabolite measurements'.
- The analysis goal is testing predefined, a priori metabolite pathway sets via overrepresentation
  instead of data-derived correlation structure.
- The normalized dataset has fewer than min_module_size=5 correlated metabolites per
  candidate module or lacks sufficient sample size to estimate biweight midcorrelations.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MetaboDiff
  role: Provides diss_matrix, identify_modules (dynamic branch cutting, min_module_size=5),
    name_modules, calculate_MS, MS_plot and MOI_plot for module detection and trait
    association
  repo: http://github.com/andreasmock/MetaboDiff/
- name: WGCNA
  role: Dependency used to derive the metabolic correlation network and detect co-abundance
    modules
- name: R
  role: Runtime environment (R version 4.0.2 or higher) in which the MetaboDiff module
    workflow is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:5c09284ccae45f0dc5e56505f85a5b6fe79aa2143beb924025311f0f0802b101
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# wgcna-module-detection-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies data-derived metabolic correlation modules from a metabolite measurement table using WGCNA in MetaboDiff, then relates module significance and module membership to sample traits. WGCNA needs no a priori metabolite sets and accounts for interaction topology, unlike standard pathway overrepresentation.

## When to use

Use when a normalized metabolite measurement table (e.g. Metabolon®-style data with SUB_PATHWAY annotations) should be partitioned into co-abundance modules and tested against sample traits such as tumor vs normal status or gender, without predefined metabolite sets.

## When NOT to use

- Input is raw mass-spectrometry spectra rather than a processed table of metabolite measurements — MetaboDiff explicitly 'starts off with the table of metabolite measurements'.
- The analysis goal is testing predefined, a priori metabolite pathway sets via overrepresentation instead of data-derived correlation structure.
- The normalized dataset has fewer than min_module_size=5 correlated metabolites per candidate module or lacks sufficient sample size to estimate biweight midcorrelations.

## Inputs

- MetaboDiff MultiAssayExperiment object with a processed (vsn-normalized, kNN-imputed at 0.4 cutoff) metabolite measurement table
- Metabolite significance p-values from diff_test for the sample trait factor (e.g. tumor_normal)
- Metabolon®-style metabolite annotation including SUB_PATHWAY

## Outputs

- Metabolic correlation modules named by dominant SUB_PATHWAY (e.g. module 2 = Creatine metabolism / Glutathione metabolism)
- Module significance (MS) measures per sample trait
- MS_plot and MOI_plot graphics (PNG) and a module significance results table

## How to apply

Start from a processed MultiAssayExperiment (vsn-normalized, imputed) with metabolite significance p-values from diff_test for the group factor of interest. Build a biweight midcorrelation-based dissimilarity matrix with diss_matrix using soft threshold beta=3, then call identify_modules with min_module_size=5 and dynamic branch cutting to assign metabolites to correlation modules. Name each module via name_modules by its most abundant SUB_PATHWAY annotation (e.g. 'Creatine metabolism', 'Glutathione metabolism'). Quantify trait association with calculate_MS for the factor (e.g. tumor_normal, random_gender), and render MS_plot (p_value_cutoff=0.05, p_adjust=FALSE) plus MOI_plot for a module of interest to inspect module membership. Rationale: WGCNA's data-derived network captures metabolite interaction topology rather than treating metabolites as independent, so modules can carry traits even when individual metabolites are not significant.

## Related tools

- **MetaboDiff** (Provides diss_matrix, identify_modules (dynamic branch cutting, min_module_size=5), name_modules, calculate_MS, MS_plot and MOI_plot for module detection and trait association) — http://github.com/andreasmock/MetaboDiff/
- **WGCNA** (Dependency used to derive the metabolic correlation network and detect co-abundance modules)
- **R** (Runtime environment (R version 4.0.2 or higher) in which the MetaboDiff module workflow is executed)

## Examples

```
install.packages("WGCNA"); devtools::install_github("andreasmock/MetaboDiff")
```

## Evaluation signals

- Dissimilarity matrix derives from biweight midcorrelation with soft threshold beta=3 (not Euclidean distance).
- All identified modules contain at least min_module_size=5 metabolites and are named by their most abundant SUB_PATHWAY annotation.
- Module 2 (Creatine metabolism / Glutathione metabolism) is significant at p_value_cutoff=0.05 with p_adjust=FALSE and shows higher abundance in normal prostate.
- MOI_plot for module 2 shows myo-inositol as the metabolite with the highest module membership.
- MS and MOI plots are rendered and exported as PNG files with module significance recorded in a table.

## Limitations

- Requires R version 4.0.2 or higher and the WGCNA package as a dependency.
- Module assignment depends on the chosen soft threshold (beta=3) and dynamic branch cutting; alternative beta values or min_module_size can change module composition.
- Module significance testing here is run with p_adjust=FALSE, so trait associations at p_value_cutoff=0.05 are not multiple-testing corrected.
- The exemplar data come from Metabolon® service output (61 prostate cancers vs 25 normal prostate specimens); other vendors' annotation schemes may not map to SUB_PATHWAY naming.

## Evidence

- [methods] the authors used the service of Metabolon® to compare the tissue metabolome of 61 prostate cancers with 25 normal prostate specimens: [evidence span withheld — anchor status: not_found]
