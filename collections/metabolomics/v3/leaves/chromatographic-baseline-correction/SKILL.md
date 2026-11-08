---
name: chromatographic-baseline-correction
description: Use when raw LC-MS EICs contain baseline drift that would distort peak-position
  detection or peak-area integration of targeted metabolites and their isotopologues,
  prior to MassSpecWavelet peak picking.
when_to_use_negative:
- When the input is already a feature/peak table without raw chromatographic traces
  — no baseline exists to correct
- When the goal is direct MS/MS library verification or data-independent acquisition
  analysis, which TarMet lists as future features rather than current capabilities
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: airPLS
  role: Baseline removal method applied to all EICs before peak detection
- name: TarMet
  role: Shiny application that orchestrates baseline correction as part of its quantification
    pipeline
  repo: https://github.com/hcji/TarMet
- name: MassSpecWavelet
  role: Downstream peak-position detection that consumes the baseline-corrected EICs
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:72ea71a03b9d08468cc0a258a0d288d06a2d5ead5731c4fa465ef7583f14b5c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chromatographic-baseline-correction

## Summary

Baseline removal of extracted ion chromatograms (EICs) using the airPLS method, a prerequisite step in TarMet's targeted metabolite and isotopologue quantification workflow that precedes MassSpecWavelet peak detection.

## When to use

Use when raw LC-MS EICs contain baseline drift that would distort peak-position detection or peak-area integration of targeted metabolites and their isotopologues, prior to MassSpecWavelet peak picking.

## When NOT to use

- When the input is already a feature/peak table without raw chromatographic traces — no baseline exists to correct
- When the goal is direct MS/MS library verification or data-independent acquisition analysis, which TarMet lists as future features rather than current capabilities

## Inputs

- Extracted ion chromatograms (EICs) from centWave-processed LC-MS data (e.g. MM14 dataset chromatograms)
- Target metabolite list with candidate adducts (M+H, M+Na, M+K) or 13C6-glucose tracer isotopologues

## Outputs

- Baseline-corrected EICs ready for MassSpecWavelet peak-position detection
- Cleaned chromatographic traces feeding into PAFFT retention-time alignment and peak-area integration

## How to apply

In the TarMet workflow, after loading centWave chromatograms (e.g. the MM14 dataset from msbi.ipb-halle.de/msbi/centwave or AssayR cellular metabolites data) and extracting EICs for each target metabolite/isotopologue, apply the airPLS method to remove the baseline of each EIC. This is done before peak positions are detected with the MassSpecWavelet package, because residual baseline would cause false peak positions and inaccurate integration. The corrected EICs then feed into PAFFT retention-time alignment across samples. The rationale is that airPLS adapts the penalty of the fitted baseline so that only non-peak regions are smoothed, preserving sharp chromatographic peaks needed for isotopologue similarity scoring via enviPat theoretical patterns.

## Related tools

- **airPLS** (Baseline removal method applied to all EICs before peak detection)
- **TarMet** (Shiny application that orchestrates baseline correction as part of its quantification pipeline) — https://github.com/hcji/TarMet
- **MassSpecWavelet** (Downstream peak-position detection that consumes the baseline-corrected EICs)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- Peak positions detected by MassSpecWavelet on corrected EICs should match expected isotopologue retention times within the sample's RT tolerance
- Isotopic similarity scores computed against enviPat theoretical patterns should approach the reported averages (0.95 for M+H, 0.93 for M+Na, 0.90 for M+K on MM14)
- After PAFFT alignment, the same isotopologue should be matched across replicate samples without RT misalignment caused by residual baseline
- In the 13C6-glucose tracer dataset, quantitative isotopologue areas should show high consistency between parallel samples after correction

## Limitations

- TarMet currently does not support data-independent acquisition; raw data must come from centWave-compatible chromatograms
- airPLS parameters are not exposed in the article; tuning may be needed for very noisy or very flat baselines
- Baseline correction alone does not resolve RT shifts — PAFFT alignment is required afterwards

## Evidence

- [abstract] airPLS as baseline removal method in TarMet workflow: "baselines of all the EICs are removed by airPLS method"
- [readme] TarMet is a shiny application for targeted metabolic analyses: "TarMet is a shiny application for targeted metabolic analyses based on mass spectrometry."
- [abstract] Peak detection follows baseline correction: "Peak positions are detected by MassSpecWavelet package"
- [other] EIC extraction precedes baseline correction in MM14 workflow: [evidence span withheld — anchor status: not_found]
