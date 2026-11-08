---
name: metabolomics-parameter-trend-analysis
description: Use when comparing metabolomics feature lists acquired under different
  MS parameter settings (collision energy, ion energy, capillary voltage) to quantify
  ISF vs. real metabolic feature trends.
when_to_use_negative:
- Input is a raw mzXML or an unprocessed feature table — this skill consumes already-computed
  count/percentage records per condition, not raw spectra or peak tables
- The goal is ISF detection itself (use ISFrag's find.level1/2/3 pipeline instead
  of tabulating parameter trends)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ISFrag
  role: Produces the ISF vs. real metabolic feature counts and ISF percentages that
    this skill tabulates across MS parameter conditions
  repo: https://github.com/HuanLab/ISFrag.git
- name: R
  role: Environment in which the parameter-keyed data frame is loaded, percentages
    recomputed, trend table assembled, and trend figure rendered to PNG
  repo: https://github.com/HuanLab/ISFrag.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:96c0d8f0fba386bf44fa8987cf2eb4dd1ebafe9ad94fe013683019ff3185668a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag.git
  tools:
  - ISFrag
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-parameter-trend-analysis

## Summary

Summarizes how LC-MS MS parameters (e.g. capillary voltage, end plate offset, ion energy, collision energy) affect counts of real metabolic features versus ISF features and the ISF percentage, using R to tabulate and plot deposited feature-count records.

## When to use

Use when comparing metabolomics feature lists acquired under different MS parameter settings (collision energy, ion energy, capillary voltage) to quantify ISF vs. real metabolic feature trends.

## When NOT to use

- Input is a raw mzXML or an unprocessed feature table — this skill consumes already-computed count/percentage records per condition, not raw spectra or peak tables
- The goal is ISF detection itself (use ISFrag's find.level1/2/3 pipeline instead of tabulating parameter trends)

## Inputs

- Deposited or supplementary feature-count records keyed by MS parameter condition (capillary voltage, end plate offset, ion energy, collision energy)
- Per-condition columns: real metabolic feature counts, ISF feature counts, ISF percentage

## Outputs

- Trend table summarizing feature counts and ISF percentages across MS parameter conditions
- Trend figure (PNG) with parameter condition on x-axis comparing real metabolic vs. ISF feature counts and ISF percentage
- Recomputed ISF percentage per parameter condition for validation

## How to apply

Retrieve the deposited or supplementary records containing feature counts and ISF percentages for each MS parameter condition (e.g. collision energy levels) and load them into R as a data frame keyed by parameter setting, with columns for real metabolic features, ISF features, and ISF percentage. Compute the ISF percentage relative to real metabolic features for each condition to confirm the deposited values. Then reproduce the trend table summarizing counts and percentages across the parameter conditions, as the ISFrag study reports that while increasing energies can increase the number of real metabolic features and ISF features, the percentage of ISF features might not necessarily increase. Finally, render a trend figure with the parameter condition on the x-axis comparing feature counts and ISF percentage, and save it as a PNG. Validate by checking that reproduced counts and percentages match the deposited values and that the trend shows both feature classes rising with energies while the ISF percentage does not necessarily increase.

## Related tools

- **ISFrag** (Produces the ISF vs. real metabolic feature counts and ISF percentages that this skill tabulates across MS parameter conditions) — https://github.com/HuanLab/ISFrag.git
- **R** (Environment in which the parameter-keyed data frame is loaded, percentages recomputed, trend table assembled, and trend figure rendered to PNG) — https://github.com/HuanLab/ISFrag.git

## Examples

```
install_github("HuanLab/ISFrag")
```

## Evaluation signals

- Recomputed ISF percentage (ISF features / real metabolic features) for each parameter condition matches the deposited values
- Trend table rows align one-to-one with the tested MS parameter conditions (e.g. collision energy settings)
- Trend figure shows both real metabolic and ISF feature counts increasing with parameter values while ISF percentage does not necessarily increase
- Output figure saved as PNG and trends qualitatively match the published ISFrag parameter-effect finding

## Limitations

- Increasing MS parameter values can increase both feature classes, but the ISF percentage may not necessarily increase — the trend must be reported as such rather than assumed monotonic
- ISF-level accuracy is method-dependent: ISFrag achieved 100% correct recognition of level 1 ISF features and over 80% for level 2 ISF features when tested on metabolite standards, so count-based trends inherit that accuracy profile

## Evidence
