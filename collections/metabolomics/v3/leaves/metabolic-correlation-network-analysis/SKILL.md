---
name: metabolic-correlation-network-analysis
description: Use when you have a table of metabolite measurements plus sample trait
  metadata and want to group metabolites into correlation modules (accounting for
  interaction topology) and test whether modules associate with traits like tumor_normal
  or gender.
when_to_use_negative:
- Input is raw mass-spectrometry spectra rather than a table of metabolite measurements
  — MetaboDiff starts off from the measurement table
- The goal is only a two-group differential test (Student's T-Test) or multi-group
  ANOVA on individual metabolites without network/module analysis
- You have predefined, curated metabolite sets (e.g. SMPDB pathway enrichment as the
  primary analysis) rather than wanting data-derived, topology-aware module detection
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0091
tools:
- name: MetaboDiff
  role: R package providing diss_matrix, identify_modules, name_modules, calculate_MS,
    MS_plot and MOI_plot to build and explore the metabolic correlation network
  repo: http://github.com/andreasmock/MetaboDiff/
- name: WGCNA
  role: Dependency supplying weighted gene co-expression/correlation network machinery
    underlying module identification in MetaboDiff
- name: devtools
  role: Used to install MetaboDiff from GitHub via install_github("andreasmock/MetaboDiff")
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:85035303e8fd80e04da95a9b88e3911d6f47be44a749da6adc7969650ac9b9b6
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
  - devtools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# metabolic-correlation-network-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a data-derived metabolic correlation network with MetaboDiff/WGCNA to identify metabolic correlation modules without a priori metabolite sets, then relates module significance and module of interest (MOI) measures to sample traits such as tumor vs normal status.

## When to use

Use when you have a table of metabolite measurements plus sample trait metadata and want to group metabolites into correlation modules (accounting for interaction topology) and test whether modules associate with traits like tumor_normal or gender.

## When NOT to use

- Input is raw mass-spectrometry spectra rather than a table of metabolite measurements — MetaboDiff starts off from the measurement table
- The goal is only a two-group differential test (Student's T-Test) or multi-group ANOVA on individual metabolites without network/module analysis
- You have predefined, curated metabolite sets (e.g. SMPDB pathway enrichment as the primary analysis) rather than wanting data-derived, topology-aware module detection

## Inputs

- Table of metabolite measurements (metabolites x samples), e.g. the Metabolon®-profiled prostate tissue data (61 cancers vs 25 normal specimens)
- Sample trait metadata (e.g. tumor_normal group factor, gender)
- Metabolite annotation from the Small Molecular Pathway Database (SMPDB), including SUB_PATHWAY
- MultiAssayExperiment object created with create_mae

## Outputs

- Metabolic correlation modules named by most abundant SUB_PATHWAY (e.g. module 2 = Creatine metabolism/Glutathione metabolism)
- Module significance (MS) measures per trait from calculate_MS
- MS_plot showing module significance vs trait association (p_value_cutoff=0.05, p_adjust=FALSE)
- MOI_plot showing module of interest membership per metabolite (e.g. myo-inositol highest in module 2)

## How to apply

Start from a normalized metabolite measurement table (e.g. after variance stabilizing normalization, vsn) stored in a MultiAssayExperiment object created with create_mae. Construct a biweight midcorrelation-based dissimilarity matrix with diss_matrix using a soft threshold of beta=3, then call identify_modules with min_module_size=5 and dynamic branch cutting, and name modules by their most abundant SUB_PATHWAY annotation using name_modules. Compute module significance (MS) for each trait (e.g. tumor_normal, random_gender) with calculate_MS and render the MS_plot with p_value_cutoff=0.05 and p_adjust=FALSE to flag significant modules. For a significant module (e.g. module 2, Creatine metabolism/Glutathione metabolism), render the MOI_plot to inspect module membership of individual metabolites, where the metabolite with the highest membership is the eigenmetabolite representative (myo-inositol in the example). The rationale is that WGCNA requires no a priori metabolite sets and captures interaction topology, so modules can be related directly to sample traits.

## Related tools

- **MetaboDiff** (R package providing diss_matrix, identify_modules, name_modules, calculate_MS, MS_plot and MOI_plot to build and explore the metabolic correlation network) — http://github.com/andreasmock/MetaboDiff/
- **WGCNA** (Dependency supplying weighted gene co-expression/correlation network machinery underlying module identification in MetaboDiff)
- **devtools** (Used to install MetaboDiff from GitHub via install_github("andreasmock/MetaboDiff"))

## Examples

```
install.packages("WGCNA"); library("devtools"); install_github("andreasmock/MetaboDiff")
```

## Evaluation signals

- The dissimilarity matrix was built with biweight midcorrelation and soft threshold beta=3 before module identification
- Modules were identified with min_module_size=5 using dynamic branch cutting, and each module is named by its most abundant SUB_PATHWAY annotation
- With p_value_cutoff=0.05 and p_adjust=FALSE, module 2 (Creatine metabolism/Glutathione metabolism) is significant for tumor_normal and shows higher abundance in normal prostate
- The MOI_plot for module 2 shows myo-inositol with the highest module membership (eigenmetabolite)
- The calculated MS table contains significance measures for both tumor_normal and random_gender traits

## Limitations

- The network analysis requires a table of metabolite measurements as starting point; it does not process raw spectra
- MetaboDiff requires R version 4.0.2 or higher
- Module naming depends on SUB_PATHWAY annotation quality (annotation retrieved from SMPDB), so unannotated or poorly annotated metabolites can mislabel modules
- Module significance depends on trait/group definition; p_adjust=FALSE in the example means multiple-testing correction is not applied at the module-plot level

## Evidence

- [abstract] As a key functionality, MetaboDiffs offers the exploration of sample traits in a data-derived metabolic correlation network: "As a key functionality, MetaboDiffs offers the exploration of sample traits in a data-derived metabolic correlation network"
- [methods] The function `create_mae` merges all objects into a so called `MultiAssayExperiment` object: "The function `create_mae` merges all objects into a so called `MultiAssayExperiment` object"
- [readme] The MetaboDiff packages aims to provide a low-level entry to differential metabolomic analysis with R by starting off with the table of metabolite measurements: "The MetaboDiff packages aims to provide a low-level entry to differential metabolomic analysis with R by starting off with the table of metabolite measurements"
