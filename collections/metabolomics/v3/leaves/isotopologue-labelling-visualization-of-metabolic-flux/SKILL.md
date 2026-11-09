---
name: isotopologue-labelling-visualization-of-metabolic-flux
description: Use when TarMet has produced a peak-area table of 13C isotopologues for
  labelled samples (e.g. MRC5 cells labelled 5 or 60 min with 13C6-glucose) and the
  goal is to visualize labelling patterns across metabolites and assess parallel-sample
  consistency.
when_to_use_negative:
- When the underlying peak areas have not yet been extracted and aligned by TarMet's
  tracer workflow (EIC extraction, airPLS baseline removal, MassSpecWavelet peak detection,
  PAFFT alignment) — the visualization depends on that quantification table.
- When the goal is untargeted discovery of labelling patterns rather than visualization
  of a predefined target metabolite list of glucose-pathway metabolites.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TarMet
  role: Reactive shiny GUI application that compiles the isotopologue quantification
    table and renders the stacked-bar isotopologue distribution figure for the tracer
    workflow
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Underlying framework for TarMet's reactive GUI, driving the visualization
    from the quantification table
- name: plotly
  role: Listed in TarMet's installation instructions as a dependency for the GUI's
    interactive plotting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:3604bbf9379de48b0da7a6ea5eb467b2535bbcbf36ec15a812eab38f3e8b1440
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - enviPat
  - plotly
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# isotopologue-labelling-visualization-of-metabolic-flux

## Summary

Render the stable isotope tracer result as a stacked-bar isotopologue distribution figure (TarMet Fig. 2) so that 13C labelling patterns across glucose-pathway metabolites and the consistency of parallel samples become visually verifiable.

## When to use

Use when TarMet has produced a peak-area table of 13C isotopologues for labelled samples (e.g. MRC5 cells labelled 5 or 60 min with 13C6-glucose) and the goal is to visualize labelling patterns across metabolites and assess parallel-sample consistency.

## When NOT to use

- When the underlying peak areas have not yet been extracted and aligned by TarMet's tracer workflow (EIC extraction, airPLS baseline removal, MassSpecWavelet peak detection, PAFFT alignment) — the visualization depends on that quantification table.
- When the goal is untargeted discovery of labelling patterns rather than visualization of a predefined target metabolite list of glucose-pathway metabolites.

## Inputs

- TarMet isotopologue peak-area quantification table (isotopologue intensities per metabolite per sample, exportable as Table S3) from 13C6-glucose-labelled samples
- Sample metadata distinguishing the 5 min and 60 min labelled parallel samples

## Outputs

- Stacked-bar figure showing the labelling pattern across glucose-pathway metabolites (corresponding to Fig. 2 in the article)
- Visual verification of parallel-sample consistency for each isotopologue

## How to apply

Take the automated isotopologue peak-area quantification table TarMet produced for each glucose-pathway metabolite in the 5 min and 60 min 13C6-glucose-labelled MRC5 fibroblast samples, and compile each metabolite's isotopologue areas into a stacked-bar chart in which the bars are the metabolites and the stacked segments are the isotopologue intensities. Keep the parallel samples together in the figure so that the consistency between replicates is directly visible, and rely on the reactive shiny GUI of TarMet to drive the rendering from the quantification table rather than hand-copying values. Judge the visualization by whether the 60 min labelled samples show clearly higher 13C incorporation than the 5 min samples across glucose-pathway metabolites — early pathway metabolites predominantly labelled, later ones unlabeled — and whether the stacked proportions match the exported Table S3 values.

## Related tools

- **TarMet** (Reactive shiny GUI application that compiles the isotopologue quantification table and renders the stacked-bar isotopologue distribution figure for the tracer workflow) — https://github.com/hcji/TarMet
- **shiny** (Underlying framework for TarMet's reactive GUI, driving the visualization from the quantification table)
- **plotly** (Listed in TarMet's installation instructions as a dependency for the GUI's interactive plotting)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- The 60 min labelled samples show clearly higher 13C incorporation in glucose-pathway metabolites than the 5 min samples.
- Early pathway metabolites appear predominantly labelled while later ones appear unlabeled, consistent with metabolism rules.
- Values for parallel samples are consistent (high consistency across replicates visible in the figure).
- The stacked segments per metabolite sum to the exported Table S3 peak-area values.
- Figure corresponds to the Fig. 2 pattern reported for the AssayR cellular metabolites dataset.

## Limitations

- Visualization is only as good as the upstream isotopologue quantification; TarMet's stated future improvements (data-independent acquisition support, MS/MS library verification) are not yet available.
- The figure reflects the target metabolite list provided to TarMet; metabolites outside the glucose-pathway list will not appear.
- Isotopologue detection is bounded by the maximum 13C considered (max 13C = 6 for 13C6-glucose).

## Evidence

- [results] Running TarMet's tracer workflow on the cellular metabolites dataset yields quantitative results for all isotopologues whose parallel samples show high consistency: [evidence span withheld — anchor status: not_found]
- [results] the stacked-bar figure (Fig. 2) shows early pathway metabolites predominantly labelled and later ones unlabeled, consistent with metabolism rules: [evidence span withheld — anchor status: not_found]
