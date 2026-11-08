---
name: module-eigenmetabolite-interpretation
description: Use when a WGCNA-derived metabolic correlation module (e.g. module 2,
  Creatine/Glutathione metabolism) must be related to sample traits and interpreted
  through its eigenmetabolite / module membership ranking.
when_to_use_negative:
- No correlation modules have been identified yet — module construction (diss_matrix,
  identify_modules, name_modules) must precede MS/MOI interpretation
- The goal is single-metabolite differential testing (Student's T-Test/ANOVA with
  Benjamini-Hochberg correction), not module-to-trait association
- Input lacks metabolite pathway annotation (SUB_PATHWAY), so modules cannot be named
  by their most abundant pathway
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetaboDiff
  role: Provides calculate_MS, MS_plot and MOI_plot functions to relate metabolic
    correlation modules to sample traits and identify each module's highest-module-membership
    metabolite
  repo: http://github.com/andreasmock/MetaboDiff/
- name: WGCNA
  role: Underlying weighted correlation network analysis used to derive the data-driven
    modules whose eigenmetabolites are interpreted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:178a218b2d520e4abb7d8109edc34ceeb38c6e58a6cbbbfa5a4da30c738d0246
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# module-eigenmetabolite-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets a metabolic correlation module from MetaboDiff's WGCNA network analysis by rendering its module significance (MS_plot) and module-of-interest (MOI_plot) plots, so that module-level traits (e.g. tumor vs normal abundance) and the module's driver metabolite (highest module membership, e.g. myo-inositol in the Creatine/Glutathione module) can be identified.

## When to use

Use when a WGCNA-derived metabolic correlation module (e.g. module 2, Creatine/Glutathione metabolism) must be related to sample traits and interpreted through its eigenmetabolite / module membership ranking.

## When NOT to use

- No correlation modules have been identified yet — module construction (diss_matrix, identify_modules, name_modules) must precede MS/MOI interpretation
- The goal is single-metabolite differential testing (Student's T-Test/ANOVA with Benjamini-Hochberg correction), not module-to-trait association
- Input lacks metabolite pathway annotation (SUB_PATHWAY), so modules cannot be named by their most abundant pathway

## Inputs

- MetaboDiff MultiAssayExperiment object (e.g. met_example dataset) with metabolite measurements and SUB_PATHWAY annotation
- Differential analysis p-values for the group factor from diff_test
- WGCNA metabolic correlation modules from diss_matrix(beta=3) + identify_modules(min_module_size=5)
- Sample trait factor (e.g. tumor_normal, random_gender)

## Outputs

- MS_plot PNG showing module significance per trait (p_value_cutoff=0.05, p_adjust=FALSE)
- MOI_plot PNG ranking metabolites of the chosen module by module membership
- Table of module significance results (e.g. module 2 significant, higher in normal prostate, myo-inositol highest module membership)

## How to apply

After differential testing of the group factor (e.g. tumor_normal via diff_test) and module identification, compute module significance measures with calculate_MS for each trait (tumor_normal and random_gender in the example dataset). Render the MS_plot with p_value_cutoff=0.05 and p_adjust=FALSE to confirm module significance (module 2 is significant with higher abundance in normal prostate). Then render the MOI_plot for the module of interest to rank metabolites by module membership; the top-ranked metabolite acts as the module's eigenmetabolite-like representative (myo-inositol for module 2). Modules are built from a biweight midcorrelation dissimilarity matrix with soft threshold beta=3 and dynamic branch cutting with min_module_size=5, named by the most abundant SUB_PATHWAY annotation. Save plots as PNG and tabulate module significance results.

## Related tools

- **MetaboDiff** (Provides calculate_MS, MS_plot and MOI_plot functions to relate metabolic correlation modules to sample traits and identify each module's highest-module-membership metabolite) — http://github.com/andreasmock/MetaboDiff/
- **WGCNA** (Underlying weighted correlation network analysis used to derive the data-driven modules whose eigenmetabolites are interpreted)

## Examples

```
met_example <- identify_modules(met_example, min_module_size = 5); met_example <- calculate_MS(met_example, factor = "tumor_normal"); MS_plot(met_example, p_value_cutoff = 0.05, p_adjust = FALSE); MOI_plot(met_example, module = 2)
```

## Evaluation signals

- Module 2 (Creatine metabolism / Glutathione metabolism) is significant for the tumor_normal trait at p_value_cutoff=0.05 with p_adjust=FALSE
- Module 2 shows higher abundance in normal prostate specimens than in tumors
- MOI_plot ranks myo-inositol as the metabolite with the highest module membership in module 2
- random_gender trait serves as a negative control — module significance should not be significant for it
- Both MS_plot and MOI_plot are successfully saved as PNG files

## Limitations

- Modules require a minimum size of 5 metabolites (min_module_size=5) and depend on the soft threshold beta=3 for the biweight midcorrelation network
- Module interpretation depends on data-derived topology rather than a priori metabolite sets, so module composition shifts with the input dataset
- MS_plot in the example uses p_adjust=FALSE, i.e. unadjusted p-values, so significance calls are not multiple-testing corrected

## Evidence
