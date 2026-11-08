---
name: biweight-midcorrelation-network-construction
description: Use when you have a table of metabolite measurements (e.g. Metabolon-style
  SUB_PATHWAY-annotated profiles) with a group factor and want data-driven metabolic
  modules whose significance can be related to sample traits.
when_to_use_negative:
- Your goal is pathway over-representation against curated, a priori metabolite sets
  — WGCNA modules are data-derived and do not require predefined sets, but will not
  recapitulate database-defined pathway membership
- Input is not a metabolite measurement table (MetaboDiff is designed to start from
  the table of metabolite measurements, not raw spectra)
- The metabolite measurement table has extensive missingness beyond the recommended
  40% (0.4) imputation cutoff and cannot be reliably imputed by k-nearest neighbor
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MetaboDiff
  role: R package providing diss_matrix, identify_modules, name_modules, calculate_MS,
    MS_plot and MOI_plot for correlation network construction and module-trait association
  repo: http://github.com/andreasmock/MetaboDiff/
- name: WGCNA
  role: Dependency installed for the weighted correlation network analysis underlying
    module detection
- name: R
  role: Runtime environment; MetaboDiff requires R version 4.0.2 or higher
- name: MultiAssayExperiment
  role: Container object (created by create_mae) holding the metabolite measurements
    fed into the network analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:f3d944390da04452dbe3a1c3a83eebb34f4f38ed95d102dc4e792f2424e6c1d9
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
  - MultiAssayExperiment
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# biweight-midcorrelation-network-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a data-derived metabolic correlation network from a metabolite measurement table using WGCNA biweight midcorrelation, then detects metabolic correlation modules by dynamic branch cutting — MetaboDiff's key functionality for exploring sample traits without a priori metabolite sets.

## When to use

Use when you have a table of metabolite measurements (e.g. Metabolon-style SUB_PATHWAY-annotated profiles) with a group factor and want data-driven metabolic modules whose significance can be related to sample traits.

## When NOT to use

- Your goal is pathway over-representation against curated, a priori metabolite sets — WGCNA modules are data-derived and do not require predefined sets, but will not recapitulate database-defined pathway membership
- Input is not a metabolite measurement table (MetaboDiff is designed to start from the table of metabolite measurements, not raw spectra)
- The metabolite measurement table has extensive missingness beyond the recommended 40% (0.4) imputation cutoff and cannot be reliably imputed by k-nearest neighbor

## Inputs

- MetaboDiff MultiAssayExperiment object of metabolite measurements (e.g. met_example dataset, Metabolon® platform)
- Differential test results (p-values) from diff_test on a group factor such as tumor_normal
- Metabolite annotations including SUB_PATHWAY (retrievable from SMPDB) for module naming
- Parameters: soft threshold beta (3), min_module_size (5)

## Outputs

- Biweight midcorrelation dissimilarity matrix
- Named metabolic correlation modules (e.g. module 2: Creatine metabolism / Glutathione metabolism)
- Module significance measures per trait (calculate_MS results table for tumor_normal, random_gender)
- MS_plot (module vs. trait significance) and MOI_plot (module of interest) plots

## How to apply

Start from a processed MultiAssayExperiment metabolite matrix (after missing-value imputation with a 40% cutoff and variance stabilizing normalization) and the diff_test results for the group factor (e.g. tumor_normal, Student's T-Test with Benjamini-Hochberg correction). Build a biweight midcorrelation-based dissimilarity matrix with diss_matrix using soft threshold beta = 3, which accounts for interaction topology rather than assuming uncorrelated metabolites. Identify modules with identify_modules (min_module_size = 5) by dynamic branch cutting, then name each module by its most abundant SUB_PATHWAY annotation via name_modules (e.g. module 2 = Creatine metabolism / Glutathione metabolism). Quantify module significance for each trait with calculate_MS, render the MS_plot (p_value_cutoff = 0.05, p_adjust = FALSE) and the MOI_plot for the module of interest, and identify the eigenmetabolite as the metabolite with the highest module membership (e.g. myo-inositol in module 2).

## Related tools

- **MetaboDiff** (R package providing diss_matrix, identify_modules, name_modules, calculate_MS, MS_plot and MOI_plot for correlation network construction and module-trait association) — http://github.com/andreasmock/MetaboDiff/
- **WGCNA** (Dependency installed for the weighted correlation network analysis underlying module detection)
- **R** (Runtime environment; MetaboDiff requires R version 4.0.2 or higher)
- **MultiAssayExperiment** (Container object (created by create_mae) holding the metabolite measurements fed into the network analysis)

## Examples

```
install.packages("WGCNA"); install_github("andreasmock/MetaboDiff")
```

## Evaluation signals

- Dissimilarity matrix is built from biweight midcorrelation with the soft threshold beta set to 3 (not default Pearson/spearman distance)
- Modules are detected by dynamic branch cutting with min_module_size = 5, and each module is named by its most abundant SUB_PATHWAY annotation
- MS_plot at p_value_cutoff = 0.05 (p_adjust = FALSE) flags module 2 (Creatine metabolism / Glutathione metabolism) as significant for tumor_normal, with higher abundance in normal prostate
- MOI_plot for module 2 shows myo-inositol as the metabolite with the highest module membership (the module eigenmetabolite)
- Both plots are exported as PNG files and the calculate_MS module significance results are recorded in a table

## Limitations

- Module detection is data-derived: composition of modules (e.g. Creatine/Glutathione) depends on the input correlation structure and soft threshold beta, so it may not match curated pathway sets
- The workflow depends on upstream preprocessing choices — k-means cluster removal, 40% missing-value imputation cutoff, and variance stabilizing normalization (vsn) — which affect the resulting correlation network
- The MetaboDiff R package requires R version 4.0.2 or higher, and WGCNA must be installed separately

## Evidence

- [full_text] Construct a biweight midcorrelation-based dissimilarity matrix with diss_matrix using a soft threshold beta of 3.: [evidence span withheld — anchor status: not_found]
- [other] MetaboDiff identifies metabolic correlation modules using WGCNA, which does not require a priori metabolite sets, accounts for interaction topology, and enables relating modules to sample traits.: [evidence span withheld — anchor status: not_found]
- [readme] As a key functionality, MetaboDiffs offers the exploration of sample traits in a data-derived metabolic correlation network.: "As a key functionality, MetaboDiffs offers the exploration of sample traits in a data-derived metabolic correlation network."
- [methods] The `MetaboDiff` R package requires R version 4.0.2 or higher.: "The `MetaboDiff` R package requires R version 4.0.2 or higher."
