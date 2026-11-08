---
name: trend-figure-rendering
description: Use when tabulated feature counts and ISF percentages per MS parameter
  setting (e.g. collision energy levels) need to be visualized as a trend figure to
  show how ISF feature prevalence changes with instrument parameters.
when_to_use_negative:
- The goal is ISF feature recognition itself from raw mzXML or a feature table — use
  the ISFrag find.level1/2/3 pipeline instead of post-hoc trend plotting.
- Only a single MS condition is available; a trend figure needs multiple parameter
  conditions to compare counts and percentages across settings.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Language used to load count/percentage records, build the parameter-keyed
    data frame, compute ISF percentages, and render the trend figure
  repo: https://github.com/HuanLab/ISFrag.git
- name: ISFrag
  role: Source package whose deposited per-parameter feature counts and ISF percentages
    supply the data being visualized; also identifies which features are ISF vs real
    metabolic
  repo: https://github.com/HuanLab/ISFrag
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:f7475c9e8978824589999d8785f18a8fd1c092676f55aa3b359eea197d58c198
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag.git
  tools:
  - ISFrag
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# trend-figure-rendering

## Summary

Renders an R plot comparing LC-MS feature counts (real metabolic vs in-source fragment) and ISF percentages across MS parameter conditions, as used in the ISFrag study to evaluate capillary voltage, ion energy, and collision energy effects.

## When to use

Use when tabulated feature counts and ISF percentages per MS parameter setting (e.g. collision energy levels) need to be visualized as a trend figure to show how ISF feature prevalence changes with instrument parameters.

## When NOT to use

- The goal is ISF feature recognition itself from raw mzXML or a feature table — use the ISFrag find.level1/2/3 pipeline instead of post-hoc trend plotting.
- Only a single MS condition is available; a trend figure needs multiple parameter conditions to compare counts and percentages across settings.

## Inputs

- R data frame keyed by MS parameter condition with columns for real metabolic feature counts, ISF feature counts, and ISF percentage (deposited or supplementary values from the ISFrag study)
- MS parameter condition labels (e.g. capillary voltage, ion energy, or collision energy settings)

## Outputs

- Trend figure (PNG) with MS parameter condition on the x-axis and real metabolic feature counts, ISF feature counts, and ISF percentage compared
- Validated data frame of recomputed ISF percentages per condition

## How to apply

Organize the deposited or supplementary count records into an R data frame keyed by parameter condition, with columns for the number of real metabolic features, the number of ISF features, and the ISF percentage relative to real metabolic features. Recompute the ISF percentage for each condition (ISF feature count divided by real metabolic feature count) and confirm it matches the deposited values before plotting. Render a trend figure with the MS parameter condition (e.g. collision energy setting) on the x-axis, plotting both feature counts and the ISF percentage so the two trends can be compared directly. The expected pattern, grounded in the ISFrag article, is that increasing energies raises the absolute counts of both real metabolic and ISF features while the ISF percentage does not necessarily increase. Save the rendered plot as a PNG file so the reproducibility of the deposited trend table can be checked by side-by-side inspection.

## Related tools

- **R** (Language used to load count/percentage records, build the parameter-keyed data frame, compute ISF percentages, and render the trend figure) — https://github.com/HuanLab/ISFrag.git
- **ISFrag** (Source package whose deposited per-parameter feature counts and ISF percentages supply the data being visualized; also identifies which features are ISF vs real metabolic) — https://github.com/HuanLab/ISFrag

## Examples

```
df <- read.csv("isf_trend_by_CE.csv"); png("isf_trend.png"); plot(df$collision_energy, df$ISF_pct, type="b", xlab="collision energy", ylab="ISF %"); lines(df$collision_energy, df$real_count, col="blue"); dev.off()
```

## Evaluation signals

- Recomputed ISF percentage (ISF features / real metabolic features) per parameter condition matches the deposited or supplementary values.
- The figure x-axis spans all MS parameter conditions present in the input data frame, with no dropped settings.
- Both real metabolic and ISF feature counts increase with increasing energies, as stated in the article's finding.
- The ISF percentage trend does NOT necessarily increase with increasing energies — a monotonically increasing percentage would signal a plotting or data-handling error.
- Output PNG file renders without missing-value warnings and all count/percentage series are visible.

## Limitations

- Increasing MS parameter values raises counts of both real metabolic and ISF features, so absolute counts alone can mislead; the ISF percentage must be plotted alongside to avoid over-interpretation.
- The ISF percentage depends on correct upstream ISF classification — ISFrag reports 100% correct recognition of level 1 ISF features but over 80% for level 2, so percentage trends derived from level 2 calls carry that uncertainty.
- The trend reflects the specific instrument parameter sweep (capillary voltage, end plate offset, ion energy, collision energy) tested; extrapolation to other instrument platforms or parameter ranges is not supported by this data.

## Evidence
