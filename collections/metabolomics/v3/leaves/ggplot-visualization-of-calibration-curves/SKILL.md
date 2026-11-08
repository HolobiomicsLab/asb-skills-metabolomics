---
name: ggplot-visualization-of-calibration-curves
description: Use when you have run calibration-curve-based quantification (e.g. get_targeted_elaboration
  on intensity and nominal-concentration tables) and need per-molecule calibration
  curve plots for inspection or reporting.
when_to_use_negative:
- Data comes from an untargeted study (feature tables from patRoon or MS-DIAL) rather
  than targeted calibration-curve quantification
- No get_targeted_elaboration results exist — plotting without fitted regression models
  has no curve to render
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: GetFeatistics
  role: Provides plot_calibration_curves, which renders per-molecule calibration curve
    ggplots from the targeted elaboration results
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: ggplot
  role: Grammar used for the calibration-curve visualisations, allowing full customisation
    of the returned plots
- name: R
  role: Language in which the package and plotting functions run (R >= 4.3.1 per README)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:62e5e162c8fb612d2848d8a109242365949c65c6bd794d194a122c553b7a4485
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - R
  - ggplot
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ggplot visualization of calibration curves

## Summary

Renders per-molecule calibration-curve plots from GetFeatistics targeted-analysis elaboration results using the plot_calibration_curves function, with outputs fully customisable via the ggplot grammar.

## When to use

Use when you have run calibration-curve-based quantification (e.g. get_targeted_elaboration on intensity and nominal-concentration tables) and need per-molecule calibration curve plots for inspection or reporting.

## When NOT to use

- Data comes from an untargeted study (feature tables from patRoon or MS-DIAL) rather than targeted calibration-curve quantification
- No get_targeted_elaboration results exist — plotting without fitted regression models has no curve to render

## Inputs

- Elaboration results from get_targeted_elaboration (fitted regression-model summaries per molecule)
- Calibration intensity table and nominal-concentration table (e.g. from create_df_examples: df_example_targeted)
- Calculated absolute concentrations and accuracy (%) for curve points and quality controls

## Outputs

- Per-molecule calibration curve ggplot objects
- Saved plot files for verification/reporting

## How to apply

First execute the targeted elaboration: run get_targeted_elaboration on the calibration intensity and nominal-concentration tables (with internal-standard normalization) so fitted regression-model summaries exist. Then call plot_calibration_curves on the elaboration results to generate one ggplot per molecule showing its fitted calibration curve and curve/QC points. Because outputs are ggplot objects, adjust them with the ggplot grammar (themes, scales, labels) before saving to files. Verify each molecule's curve by checking that accuracy (%) values for curve points and quality controls fall in sensible ranges and that fitted regression summaries match the plotted curve.

## Related tools

- **GetFeatistics** (Provides plot_calibration_curves, which renders per-molecule calibration curve ggplots from the targeted elaboration results) — github.com/FrigerioGianfranco/GetFeatistics
- **ggplot** (Grammar used for the calibration-curve visualisations, allowing full customisation of the returned plots)
- **R** (Language in which the package and plotting functions run (R >= 4.3.1 per README))

## Examples

```
library(GetFeatistics); dfs <- create_df_examples(); res <- get_targeted_elaboration(dfs$df_example_targeted); plot_calibration_curves(res)
```

## Evaluation signals

- Each molecule in the calibration table yields its own calibration-curve ggplot
- Plotted points match the returned accuracy (%) values for curve points and quality controls
- The plotted curve is consistent with the fitted regression-model summary from get_targeted_elaboration
- Plots are valid ggplot objects and can be further modified via the ggplot grammar before saving
- Saved plot files open and correspond to the molecules in results_concentrations

## Limitations

- Plots are only meaningful downstream of the calibration-curve quantification; the package's targeted weighting options affect the fitted curve
- Requires the GetFeatistics package installed (R >= 4.3.1, Java matching R architecture); contact the maintainer if unclear per README

## Evidence

- [abstract] For targeted analyses, the package enables calibration curve-based quantification with different data weighting options.: "For targeted analyses, the package enables calibration curve-based quantification with different data weighting options."
- [abstract] output visualisations are fully customisable via the ggplot grammar: "output visualisations are fully customisable via the ggplot grammar"
- [readme] Check also the vignette guiding you through the workflow step by step: [evidence span withheld — anchor status: external_doc]
