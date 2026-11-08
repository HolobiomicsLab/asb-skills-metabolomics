---
name: r-package-vignette-reproduction
description: 'Use when verifying an installed R package against its published vignette:
  you have the package name (e.g. GetFeatistics), its bundled mock/example data, and
  the vignette URL, and the goal is to reproduce the documented workflow rather than
  analyze new data.'
when_to_use_negative:
- When the goal is to analyze real, previously unseen study data rather than verify
  that the package reproduces its documented vignette behaviour on bundled mock data.
- When the input is already a fully processed feature table and only downstream statistical
  modelling is needed — use the specific regression/QC-filtering skills instead.
- When the package under test is not an R package with a published vignette (e.g.
  CLI tools like MS-DIAL or web services); a scripted vignette reproduction is not
  applicable.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: GetFeatistics
  role: The R package whose published vignette is being reproduced; supplies transf_data,
    gentab_descr, gentab_P.t.test, gentab_FC, gentab_P.1wayANOVA_posthocTukeyHSD,
    gentab_P.2wayANOVA_posthocTukeyHSD, and the ggplot-based volcano plot function
    used in the walkthrough.
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime language (>= 4.3.1) in which the vignette is executed and all package
    calls are made.
- name: ggplot
  role: Grammar underlying the visualisation output (e.g. volcano plot) produced during
    the reproduction.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:fe5e7330fba6acd3f4f4b5ec19e054e07b57dc821077ea20dfe4784271dc7bbf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - R
  - ggplot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# r-package-vignette-reproduction

## Summary

Reproduce an R package's published vignette workflow end-to-end on the package's bundled example data, validating that the installed package, its functions, and its documented parameters behave as the article describes. For GetFeatistics this means running the targeted/untargeted mock data through QC-oriented filtering, intensity transformation, univariate statistics (t-test, fold change, one- and two-way ANOVA), and ggplot visualisation exactly as the GF_vignette walkthrough prescribes.

## When to use

Use when verifying an installed R package against its published vignette: you have the package name (e.g. GetFeatistics), its bundled mock/example data, and the vignette URL, and the goal is to reproduce the documented workflow rather than analyze new data.

## When NOT to use

- When the goal is to analyze real, previously unseen study data rather than verify that the package reproduces its documented vignette behaviour on bundled mock data.
- When the input is already a fully processed feature table and only downstream statistical modelling is needed — use the specific regression/QC-filtering skills instead.
- When the package under test is not an R package with a published vignette (e.g. CLI tools like MS-DIAL or web services); a scripted vignette reproduction is not applicable.

## Inputs

- Installed R package (e.g. GetFeatistics, R >= 4.3.1) with its bundled example/mock data (df_example_sample_data plus targeted and untargeted feature tables)
- Vignette URL or local vignette document describing the step-by-step workflow (e.g. https://frigeriogianfranco.github.io/GetFeatistics/articles/GF_vignette.html)

## Outputs

- Descriptive-statistics table (gentab_descr)
- Pairwise t-test table with FDR-corrected p-values (gentab_P.t.test)
- Fold-change table (gentab_FC)
- One-way and two-way ANOVA tables with post-hoc Tukey HSD and FDR correction (gentab_P.1wayANOVA_posthocTukeyHSD, gentab_P.2wayANOVA_posthocTukeyHSD)
- Volcano plot rendering the t-test/fold-change results (ggplot-based)

## How to apply

Install the package from GitHub with devtools::install_github(..., dependencies = TRUE) and load it into R (version >= 4.3.1, Java architecture matching R). Follow the vignette step by step on the bundled mock data: join df_example_sample_data with the targeted and untargeted feature tables, apply transf_data to replace missing values, log-transform intensities and apply pareto scaling, then run gentab_descr for a descriptive table. Produce pairwise comparisons with gentab_P.t.test (with FDR correction) and gentab_FC, one- and two-way ANOVA with post-hoc Tukey HSD and FDR via gentab_P.1wayANOVA_posthocTukeyHSD and gentab_P.2wayANOVA_posthocTukeyHSD, and render the volcano plot from the t-test/fold-change results with the package's plotting function. Output all tables and the plot as files mirroring the vignette, and treat any deviation (wrong table shape, missing QC-filtering behaviour, unrenderable plot) as a signal that installation or the package state is incorrect.

## Related tools

- **GetFeatistics** (The R package whose published vignette is being reproduced; supplies transf_data, gentab_descr, gentab_P.t.test, gentab_FC, gentab_P.1wayANOVA_posthocTukeyHSD, gentab_P.2wayANOVA_posthocTukeyHSD, and the ggplot-based volcano plot function used in the walkthrough.) — github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime language (>= 4.3.1) in which the vignette is executed and all package calls are made.)
- **ggplot** (Grammar underlying the visualisation output (e.g. volcano plot) produced during the reproduction.)

## Examples

```
Rscript -e 'library(GetFeatistics); data(df_example_sample_data); td <- transf_data(df_example_sample_data); print(gentab_descr(td)); print(gentab_P.t.test(td)); print(gentab_FC(td))'
```

## Evaluation signals

- The installed package loads and the vignette's bundled mock data (df_example_sample_data, targeted and untargeted feature tables) are available without errors.
- transf_data output shows missing values replaced, log-transformed intensities, and pareto-scaled features as the vignette describes.
- Each gentab_* call returns a table with the expected shape/content (e.g. FDR-corrected t-test p-values, fold-change values, ANOVA tables with Tukey HSD rows).
- The volcano plot renders successfully from the gentab_P.t.test and gentab_FC results and matches the vignette's ggplot-based appearance.
- All vignette output files (tables and plots) are produced and mirror the vignette's rendered artifacts.

## Limitations

- Reproduction depends on the exact package version and its dependencies; devtools::install_github(..., dependencies = TRUE) may take considerable time and the vignette output may drift if the package is updated.
- The vignette uses bundled mock data; it does not demonstrate behaviour on real, large-scale, or messy study data, so a successful reproduction does not guarantee correct behaviour on new inputs.
- The skill is specific to R packages with published vignettes; it does not generalize to CLI tools or web services without an equivalent scripted walkthrough.

## Evidence

- [readme] Install from GitHub with devtools and run the vignette walkthrough on bundled mock data: "devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE)"
- [readme] Vignette URL is the canonical walkthrough to follow: "Check also the vignette guiding you through the workflow step by step:

https://frigeriogianfranco.github.io/GetFeatistics/articles/GF_vignette.html"
- [abstract] Article abstract: package purpose and R base: "GetFeatistics is an R-package developed to streamline the elaboration and statistical analysis of metabolomics data."
- [intro] Article intro: univariate stats and normalisation steps the vignette applies: "data intensities are pre-processed to deal with missing values and data normality"
