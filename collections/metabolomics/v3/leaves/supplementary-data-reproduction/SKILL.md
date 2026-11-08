---
name: supplementary-data-reproduction
description: Use when a task requires reproducing a deposited trend table or figure
  showing how MS parameters (e.g. capillary voltage, ion energy, collision energy)
  change the number and percentage of ISF features relative to real metabolic features.
when_to_use_negative:
- You are re-running ISFrag de novo recognition from raw mzXML files rather than reproducing
  a deposited/summarized effect table
- The input is already a finished trend figure or the deposited values cannot be retrieved,
  leaving nothing to reproduce or validate against
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ISFrag
  role: R package whose ISF recognition output (get.ISFrag.results) supplies the ISF
    feature counts and precursor-fragment relationships that are tabulated per MS
    parameter condition
  repo: https://github.com/HuanLab/ISFrag
- name: R
  role: Language used to load the deposited records, recompute ISF percentages, build
    the trend table, and render the comparison figure as PNG
  repo: https://github.com/HuanLab/ISFrag
- name: RStudio
  role: Recommended environment for running the ISFrag installation and the R reproduction/plotting
    scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:dd474d46a7e7ce3a06af429efefdd6eb52431c56976670a1273c8488101e8402
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag
  tools:
  - ISFrag
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# supplementary-data-reproduction

## Summary

Retrieves deposited/supplementary MS-parameter effect data (feature counts and ISF percentages per parameter condition), reproduces them in R, and renders a trend figure comparing real metabolic and ISF feature counts across settings such as collision energy.

## When to use

Use when a task requires reproducing a deposited trend table or figure showing how MS parameters (e.g. capillary voltage, ion energy, collision energy) change the number and percentage of ISF features relative to real metabolic features.

## When NOT to use

- You are re-running ISFrag de novo recognition from raw mzXML files rather than reproducing a deposited/summarized effect table
- The input is already a finished trend figure or the deposited values cannot be retrieved, leaving nothing to reproduce or validate against

## Inputs

- Deposited/supplementary CSV or table of feature counts and ISF percentages per MS parameter condition
- ISFrag result feature table (complete feature table with ISF relationship annotated in additional columns, or a detailed ISF precursor-fragment relationship dataframe)
- R environment (R >= 4.0.0) with the ISFrag package installed

## Outputs

- Reproduced trend table summarizing real metabolic feature counts, ISF feature counts, and ISF percentage across parameter conditions
- PNG trend figure comparing feature counts and ISF percentage as a function of the MS parameter condition
- Validation check that reproduced counts and percentages match the deposited values

## How to apply

Load the supplementary/deposited records with R and assemble them into a data frame keyed by MS parameter condition (e.g. collision energy setting), with columns for real metabolic features, ISF features, and ISF percentage. Recompute the ISF percentage as the ratio of ISF features to real metabolic features for each condition to confirm the deposited values before building any figure. Reproduce the trend table summarizing counts and percentages across parameter conditions, then render a comparison plot with the parameter condition on the x-axis and feature counts versus ISF percentage overlaid, saving it as PNG. The key validation is the paper's finding: increasing energies can increase the number of both real metabolic and ISF features while the percentage of ISF features might not necessarily increase — verify the reproduced numbers match the deposited values and that the trend reflects this non-monotonic percentage behavior.

## Related tools

- **ISFrag** (R package whose ISF recognition output (get.ISFrag.results) supplies the ISF feature counts and precursor-fragment relationships that are tabulated per MS parameter condition) — https://github.com/HuanLab/ISFrag
- **R** (Language used to load the deposited records, recompute ISF percentages, build the trend table, and render the comparison figure as PNG) — https://github.com/HuanLab/ISFrag
- **RStudio** (Recommended environment for running the ISFrag installation and the R reproduction/plotting scripts)

## Examples

```
install.packages("devtools"); devtools::install_github("HuanLab/ISFrag"); library(ISFrag)
```

## Evaluation signals

- Recomputed ISF percentage for each parameter condition equals the deposited percentage (within rounding)
- Reproduced counts of real metabolic features and ISF features match the deposited values exactly
- Trend shows both real metabolic and ISF feature counts increasing with energies, while the ISF percentage does not necessarily increase (non-monotonic check)
- Trend figure is written as a PNG with the parameter condition on the x-axis and both counts and percentage represented

## Limitations

- ISFrag was validated with metabolite standards: 100% correct recognition of level 1 ISF features but only over 80% for level 2 features, so reproduced level 2 counts inherit this uncertainty
- Percentage of ISF features is not monotonic with increasing MS parameters, so simple linear trend descriptions of the table will be misleading
- The skill requires the deposited supplementary records to be retrievable; if only raw data is available, the full ISFrag workflow (XCMS feature extraction, find.level1/2/3, get.ISFrag.results) must be run first

## Evidence
