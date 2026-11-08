---
name: isf-feature-count-tabulation
description: Use when deposit/supplementary data contain feature counts and ISF percentages
  per MS parameter condition and the goal is to reproduce a table comparing real metabolic
  features vs ISF features across parameter settings.
when_to_use_negative:
- The input is already an annotated ISF result feature table from get.ISFrag.results()
  and the task is ISF identification rather than cross-condition count/percentage
  tabulation.
- Only a single MS parameter condition is available — percentage trends across parameter
  settings cannot be reproduced from one condition.
- The analysis goal is adduct/isotope annotation rather than comparing ISF feature
  percentages across MS parameter values.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ISFrag
  role: R package used to generate ISF feature counts per parameter condition via
    find.level3/find.level2/find.level1 and get.ISFrag.results, summarized into the
    tabulation
  repo: https://github.com/HuanLab/ISFrag.git
- name: R
  role: Language in which the count/percentage records are loaded, organized into
    a data frame, and the trend table and PNG figure are rendered
  repo: https://github.com/HuanLab/ISFrag.git
- name: xcms
  role: XCMS.featuretable extracts MS1 features from mzXML files acquired under each
    parameter condition before ISF identification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:584d3f691145503a3bfd5876fe5c2265779d0b8c0eb41682479420a79d2b0f11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag.git
  tools:
  - ISFrag
  - R
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# isf-feature-count-tabulation

## Summary

Tabulate ISF feature counts and percentages relative to real metabolic features across MS parameter conditions (e.g. capillary voltage, ion energy, collision energy) using ISFrag output, then render a trend comparison to show how parameter changes shift feature composition.

## When to use

Use when deposit/supplementary data contain feature counts and ISF percentages per MS parameter condition and the goal is to reproduce a table comparing real metabolic features vs ISF features across parameter settings.

## When NOT to use

- The input is already an annotated ISF result feature table from get.ISFrag.results() and the task is ISF identification rather than cross-condition count/percentage tabulation.
- Only a single MS parameter condition is available — percentage trends across parameter settings cannot be reproduced from one condition.
- The analysis goal is adduct/isotope annotation rather than comparing ISF feature percentages across MS parameter values.

## Inputs

- Deposited or supplementary records of feature counts and ISF percentages per MS parameter condition (e.g. collision energy settings)
- mzXML files (fullscan/DDA/DIA) acquired under each MS parameter condition for ISFrag feature extraction
- CSV feature table (m/z, rt, rtmin, rtmax, per-sample intensity columns) from MS-DIAL, MZmine2, or XCMS as alternative ISFrag input

## Outputs

- Data frame keyed by MS parameter setting with columns for real metabolic feature count, ISF feature count, and ISF percentage
- Trend table summarizing counts and percentages across parameter conditions
- Trend figure (PNG) plotting feature counts and ISF percentage against parameter condition

## How to apply

Retrieve the deposited or supplementary records containing, for each MS parameter condition, the count of real metabolic features, the count of ISF features, and the ISF percentage relative to real metabolic features. Load these records in R and organize them into a data frame keyed by parameter setting, with columns for real metabolic feature count, ISF feature count, and ISF percentage; recompute the ISF percentage as ISF count divided by real metabolic feature count to confirm the deposited values. The ISF feature counts themselves should come from running the ISFrag pipeline (XCMS.featuretable or custom.featuretable input, then find.level3()/find.level2()/find.level1() and get.ISFrag.results()) on mzXML files acquired under each parameter condition. Assemble a summary table ordered by parameter value and render a trend figure (parameter condition on the x-axis, feature counts and ISF percentage compared) saved as a PNG. Validate by checking that recomputed counts and percentages match the deposited values and that the trend reflects the article's finding: increasing energies (or other MS parameters) can increase both real metabolic and ISF feature counts, while the ISF percentage may not necessarily increase.

## Related tools

- **ISFrag** (R package used to generate ISF feature counts per parameter condition via find.level3/find.level2/find.level1 and get.ISFrag.results, summarized into the tabulation) — https://github.com/HuanLab/ISFrag.git
- **R** (Language in which the count/percentage records are loaded, organized into a data frame, and the trend table and PNG figure are rendered) — https://github.com/HuanLab/ISFrag.git
- **xcms** (XCMS.featuretable extracts MS1 features from mzXML files acquired under each parameter condition before ISF identification)

## Examples

```
library(ISFrag); xcmsFT <- XCMS.featuretable(MS1directory = MS1directory, type = "single", peakwidth = c(5,20))
```

## Evaluation signals

- Recomputed ISF percentages (ISF count / real metabolic feature count per condition) match the deposited or supplementary values exactly.
- Feature counts for both real metabolic and ISF features are present and non-decreasing (or at least non-zero) for every parameter condition included in the table.
- The trend table ordering is consistent (monotonic in parameter value) and every parameter condition from the source data appears as a row.
- The rendered PNG opens and shows parameter condition on the x-axis with both feature counts and ISF percentage comparable as in the article's figure.
- Trend direction agrees with the article's finding: counts may rise with increased energies while the ISF percentage does not necessarily increase.

## Limitations

- Increasing MS parameter values does not guarantee a higher ISF percentage even when both real and ISF feature counts increase — the percentage trend is not monotonic.
- CAMERA adduct and isotope annotation can only be used for XCMS-only ISFrag analysis, so custom feature table workflows limit annotation options.
- ISF percentage depends on correct ISF level assignment; ISFrag achieved 100% correct recognition of level 1 ISFs but over 80% (not perfect) for level 2 ISFs on metabolite standards.

## Evidence
