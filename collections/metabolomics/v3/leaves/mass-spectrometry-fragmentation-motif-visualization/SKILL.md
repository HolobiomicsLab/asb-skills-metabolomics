---
name: mass-spectrometry-fragmentation-motif-visualization
description: Use when a trained topic model over MS/MS fragmentation data (Mass2Motifs
  assigned to spectra) needs interactive visual exploration before motif characterization,
  e.g. filtering motifs by degree prior to biochemical interpretation.
when_to_use_negative:
- The goal is automated annotation of motifs via spectral matching rather than interactive
  visual exploration (the paper treats visualization as a pre-characterization step)
- Input is a simple mixture with sparse, information-poor MS2 spectra, where MS2LDA
  topic modeling itself performs poorly
- Motif loadings and term distributions are already exported in the MS2LDAViz JSON
  format and only web-app viewing is needed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MS2LDA
  role: Produces the trained LDA model (ms2lda.bin), motifset.json, and topic-term/topic-prevalence
    outputs consumed by the visualization module (MS2LDA.Visualisation)
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Customization of LDAvis used to project Mass2Motifs in 2D, render prevalence-scaled
    bubbles, and rank fragment/neutral-loss terms with a lambda slider
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAViz
  role: Web-based application that consumes the bundled ms2lda_viz.json.gz rendering
    for interactive exploration and analysis of results
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:3aef5821d4a7ff8188b8d621212254d1e94f6f7ec42378ccc77008563ae30e02
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - LDAvis
  - MS2LDA
  - MS2LDAViz
  - MS2LDAvis
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-spectrometry-fragmentation-motif-visualization

## Summary

Interactive 2D exploration of LDA-derived Mass2Motifs using MS2LDAvis, a customization of LDAvis, where each motif is rendered as a bubble sized by its spectral prevalence and ranked fragment/neutral-loss terms are inspected via a relevance (lambda) slider.

## When to use

Use when a trained topic model over MS/MS fragmentation data (Mass2Motifs assigned to spectra) needs interactive visual exploration before motif characterization, e.g. filtering motifs by degree prior to biochemical interpretation.

## When NOT to use

- The goal is automated annotation of motifs via spectral matching rather than interactive visual exploration (the paper treats visualization as a pre-characterization step)
- Input is a simple mixture with sparse, information-poor MS2 spectra, where MS2LDA topic modeling itself performs poorly
- Motif loadings and term distributions are already exported in the MS2LDAViz JSON format and only web-app viewing is needed

## Inputs

- ms2lda.bin — trained LDA model binary
- motifset.json — discovered motif set with fragment and neutral-loss term distributions and document-to-spectra mapping
- LDA inference output: topic-term distributions and spectra-level topic prevalences

## Outputs

- ms2lda_viz.json.gz — gzipped JSON consumed by the MS2LDAViz web app containing every inferred Mass2Motif with its fragment/loss terms and spectra-level loadings
- Interactive 2D bubble plot of Mass2Motifs (bubble area = overall prevalence) with per-motif ranked fragment and neutral-loss terms

## How to apply

Load the trained LDA model binary (ms2lda.bin) and the motif set (motifset.json) holding fragment and neutral-loss term distributions plus the document-to-spectra mapping. Use MS2LDA.Visualisation to compute topic-term distributions and spectra-level topic prevalences from the LDA output, then adapt the LDAvis (Sievert & Shirley 2014) projection: project Mass2Motifs into 2D by inter-topic distance, draw each motif as a bubble whose area encodes overall prevalence, and expose a lambda slider to reorder fragment and neutral-loss terms per motif. In the published analysis, Mass2Motifs were inspected at a degree filter of ≥10, i.e. motifs occurring in 10 or more spectra, before further characterization. Bundle the interactive rendering into the compressed JSON (ms2lda_viz.json.gz) consumed by the MS2LDAViz web app.

## Related tools

- **MS2LDA** (Produces the trained LDA model (ms2lda.bin), motifset.json, and topic-term/topic-prevalence outputs consumed by the visualization module (MS2LDA.Visualisation)) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Customization of LDAvis used to project Mass2Motifs in 2D, render prevalence-scaled bubbles, and rank fragment/neutral-loss terms with a lambda slider) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAViz** (Web-based application that consumes the bundled ms2lda_viz.json.gz rendering for interactive exploration and analysis of results) — https://github.com/vdhooftcompmet/MS2LDA

## Evaluation signals

- ms2lda_viz.json.gz exists and is valid gzipped JSON
- The export contains every inferred Mass2Motif with its fragment/loss terms and spectra-level loadings (the paper reports 300 Mass2Motifs per data file, so the export should cover the full motif set)
- Bubble areas in the 2D projection correctly reflect each motif's overall prevalence across spectra
- The lambda relevance slider correctly reorders fragment and neutral-loss terms per motif
- Visual interpretability confirmed by expert review of the MS2LDAvis network subset shown in the SI (section S1.4)

## Limitations

- Motif degree varies widely (1 to over 200 spectra), so high-prevalence motifs can dominate the visualization while rare motifs remain hard to inspect
- Interpretation of the rendered motifs still requires manual biochemical characterization; the visualization itself does not annotate motifs
- Automated annotation of rendered motifs omits neutral losses, which are not currently supported by spectral matching tools

## Evidence

- [readme] MS2LDAViz web application for exploring results: [evidence span withheld — anchor status: external_doc]
