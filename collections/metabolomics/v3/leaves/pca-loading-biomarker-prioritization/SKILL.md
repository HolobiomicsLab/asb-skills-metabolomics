---
name: pca-loading-biomarker-prioritization
description: Use when an untargeted DI-MS or LC-MS peak table has been normalized
  in MetaboShiny (filtered, normalized, transformed, scaled) and you need to rank
  which m/z values separate a disease group from controls before annotating candidates.
when_to_use_negative:
- The data has not yet been filtered/normalized in MetaboShiny — run the normalization
  step first so m/z distributions are comparable between samples and batches
- You want to annotate all m/z values up-front — MetaboShiny performs annotation only
  on significant hits from the analysis, not on the full peak table
- Raw peak data (not yet peak-picked via XCMS/MSnbase or a MetaboAnalyst-style export)
  — MetaboShiny does not accept raw peak data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboShiny
  role: Interactive R/Shiny app that performs the dimension reduction (PCA, PLS-DA,
    t-SNE), displays loadings, and prioritizes compounds from significant m/z hits
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: R
  role: Runtime language in which MetaboShiny and its MetaboAnalystR-backed statistics
    execute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:e4f8172084e00db96228a168ff8f6d37921633ad329ee63d4d8f1fd013b3bfcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pca-loading-biomarker-prioritization

## Summary

Prioritize potential biomarker m/z values by inspecting dimension-reduction loadings (PCA or PLS-DA) after MetaboShiny normalization, so that discriminatory m/z ratios like 264, 308, 441, 561 rise to the top of the loading ranking before compound annotation. This is how the MetaboShiny paper reproduces the Mathé et al. lung cancer urine DI-MS case study.

## When to use

Use when an untargeted DI-MS or LC-MS peak table has been normalized in MetaboShiny (filtered, normalized, transformed, scaled) and you need to rank which m/z values separate a disease group from controls before annotating candidates.

## When NOT to use

- The data has not yet been filtered/normalized in MetaboShiny — run the normalization step first so m/z distributions are comparable between samples and batches
- You want to annotate all m/z values up-front — MetaboShiny performs annotation only on significant hits from the analysis, not on the full peak table
- Raw peak data (not yet peak-picked via XCMS/MSnbase or a MetaboAnalyst-style export) — MetaboShiny does not accept raw peak data

## Inputs

- MetaboLights-format or MetaboShiny-native m/z peak table (positive and/or negative mode)
- Metadata table with 'sample', 'individual' and experimental-group columns
- MetaboShiny project settings: mass spectrometer ppm error margin and m/z missing-value percentage threshold

## Outputs

- Ranked m/z loadings from PCA/PLS-DA (PC1 loadings table)
- PLS-DA group separation p-value and explained variance per component
- plsda_separation_results.csv (t-test table, PLS-DA variance/loadings, separation p-value)
- Top discriminatory m/z values ready for downstream compound annotation

## How to apply

In MetaboShiny, load the merged peak/metadata project (e.g. MetaboLights MTBLS28 urine data from 1005 patients with and without lung cancer), complete data normalization (Random Forest imputation as used for the paper's test dataset), and select the disease-vs-control comparison. Run dimension reduction via MetaboAnalystR-backed PLS-DA (and PCA/t-SNE as alternatives) on the lung cancer vs. control groups, then inspect the PC1 loadings: the paper's validation expects the four reported cancer-predictive m/z values (264, 308, 441, 561) to appear among the top loadings and the PLS-DA group separation to reach p < 0.03. Export the PLS-DA variance, loadings, and separation p-value (e.g. as plsda_separation_results.csv) and only then proceed to m/z annotation of the top hits, since MetaboShiny deliberately annotates only significant hits after the analysis rather than all m/z values.

## Related tools

- **MetaboShiny** (Interactive R/Shiny app that performs the dimension reduction (PCA, PLS-DA, t-SNE), displays loadings, and prioritizes compounds from significant m/z hits) — https://github.com/joannawolthuis/MetaboShiny
- **R** (Runtime language in which MetaboShiny and its MetaboAnalystR-backed statistics execute)

## Examples

```
docker run -p 8080:8080 -v ~/MetaboShiny/:/root/MetaboShiny/:cached --rm -it jcwolthuis/metaboshiny:latest Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"
```

## Evaluation signals

- PLS-DA separation p-value between lung cancer and control groups satisfies p < 0.03 as reported in the Mathé et al. paper
- The four known cancer-predictive m/z values (264, 308, 441, 561) rank among the top PC1 loadings
- Creatine riboside (M + H adduct) is recoverable as a putative hit at 264 m/z when the HMDB database is searched
- Post-normalization peak-value distribution plots for randomly selected m/z values and samples confirm the chosen filter/normalize/transform/scale parameters changed the data distribution as expected
- Exported results file contains a coherent t-test table, PLS-DA variance/loadings and separation p-value

## Limitations

- Missing-value imputation with Random Forest is slow — other normalization methods are expected to run substantially faster
- DI-MS routinely produces over a hundred thousand unidentified m/z values, so loading inspection is only meaningful after filtering by the user-set m/z missing-value threshold
- Database-dependent: loading-ranked hits only become interpretable metabolites after database search (35 supported databases), and MetaboShiny does not automatically update built databases
- PLS-DA loading ranking reflects the specific normalization choices (filtering method, normalization type, transformation, scaling, imputation) and should be re-checked if those parameters change

## Evidence

- [methods] The displayed results themselves are obtained from a MetaboLights dataset MTBLS28: "The displayed results themselves are obtained from a MetaboLights dataset MTBLS28"
- [methods] various dimension reduction methods such as PCA and PLS-DA, and volcano plots: "various dimension reduction methods such as PCA and PLS-DA, and volcano plots"
- [methods] This work identified 4 m/z values at MSI level 1, 264 m/z, 308 m/z, 441 m/z and 561 m/z predictive for cancer status: [evidence span withheld — anchor status: not_found]
- [intro] MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis.: "MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis."
- [methods] Missing values were imputed using Random Forest, other normalization methods are expected to run substantially faster: "Missing values were imputed using Random Forest, other normalization methods are expected to run substantially faster"
