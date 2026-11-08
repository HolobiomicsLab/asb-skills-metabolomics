---
name: lda-topic-model-visualization
description: Use when you have a trained LDA topic model over MS/MS fragmentation
  data (e.g., ms2lda.bin with motifset.json) and need interactive exploration of inferred
  topics and their fragment/neutral-loss terms before structural characterization.
when_to_use_negative:
- The analysis goal is structural characterization of already-selected motifs (annotation
  via spectral matching), not interactive exploration of the topic model output.
- Input spectra are simple or information-poor (few fragments, no ramped/stepped collision
  energy) — the paper states MS2LDA 'works best for complex mixtures where a large
  number of metabolites are fragmented and information-rich MS2 spectra are available'.
- You only need a static figure of the spectral network; the SI shows a subset network
  produced by MS2LDAvis, but the interactive module is for exploration, not final
  publication rendering.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MS2LDA
  role: Framework whose Visualisation module (MS2LDA.Visualisation) computes topic-term
    distributions and spectra-level topic prevalences from the LDA output; repo provides
    MS2LDAViz web app
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Customization of LDAvis adapted to the MS/MS domain for interactive exploration
    of Mass2Motifs
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: LDAvis
  role: 'Original topic-model visualization approach (Sievert & Shirley 2014) adapted:
    2D inter-topic projection, prevalence-proportional bubbles, lambda relevance slider'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:31c0a0fb9785924ef367543bb13e4f2cd4892d9676088a337de7e114c7385899
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - LDAvis
  - MS2LDA
  - MS2LDAvis
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lda-topic-model-visualization

## Summary

Reconstructs the MS2LDAvis interactive exploration module — a customization of LDAvis (Sievert & Shirley 2014) — so that trained LDA topic-model output (Mass2Motifs, their fragment/neutral-loss term distributions, and spectra-level topic loadings) can be inspected in a 2D inter-topic bubble plot with a lambda relevance slider for ranking terms per motif.

## When to use

Use when you have a trained LDA topic model over MS/MS fragmentation data (e.g., ms2lda.bin with motifset.json) and need interactive exploration of inferred topics and their fragment/neutral-loss terms before structural characterization.

## When NOT to use

- The analysis goal is structural characterization of already-selected motifs (annotation via spectral matching), not interactive exploration of the topic model output.
- Input spectra are simple or information-poor (few fragments, no ramped/stepped collision energy) — the paper states MS2LDA 'works best for complex mixtures where a large number of metabolites are fragmented and information-rich MS2 spectra are available'.
- You only need a static figure of the spectral network; the SI shows a subset network produced by MS2LDAvis, but the interactive module is for exploration, not final publication rendering.

## Inputs

- Trained LDA model binary (ms2lda.bin)
- Discovered motif set (motifset.json) with fragment and neutral-loss term distributions per Mass2Motif and the document-to-spectra mapping
- LDA inference output: Mass2Motifs and their assignments to spectra

## Outputs

- ms2lda_viz.json.gz — valid gzipped JSON containing every inferred Mass2Motif with its fragment/loss terms and spectra-level loadings
- Interactive 2D bubble visualization of Mass2Motifs (area = overall prevalence, positioned by inter-topic distance) with a lambda relevance slider for ranking terms
- Filtered set of Mass2Motifs with degree ≥10 (present in ≥10 spectra) for further biochemical characterization

## How to apply

Load the trained LDA model binary (ms2lda.bin) and the discovered motif set (motifset.json) holding each Mass2Motif's fragment and neutral-loss term distributions plus the document-to-spectra mapping. Use the MS2LDA Visualisation module to compute topic-term distributions and spectra-level topic prevalences from the LDA output, then adapt the LDAvis approach to the MS/MS domain: project Mass2Motifs into 2D by inter-topic distance, render each motif as a bubble whose area reflects overall prevalence, and expose a relevance (lambda) slider for ranking fragment and neutral-loss terms per motif. Bundle the interactive rendering into the compressed JSON format (ms2lda_viz.json.gz) consumed by the MS2LDAViz web app. When curating motifs for further characterization, inspect them at a degree filter of ≥10, i.e., motifs present in 10 or more spectra — the paper notes Mass2Motif degrees ranged from 1 to over 200 spectra, with 300 motifs extracted per data file.

## Related tools

- **MS2LDA** (Framework whose Visualisation module (MS2LDA.Visualisation) computes topic-term distributions and spectra-level topic prevalences from the LDA output; repo provides MS2LDAViz web app) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Customization of LDAvis adapted to the MS/MS domain for interactive exploration of Mass2Motifs) — https://github.com/vdhooftcompmet/MS2LDA
- **LDAvis** (Original topic-model visualization approach (Sievert & Shirley 2014) adapted: 2D inter-topic projection, prevalence-proportional bubbles, lambda relevance slider)

## Evaluation signals

- ms2lda_viz.json.gz exists, decompresses as valid gzip, and parses as valid JSON
- Export contains every inferred Mass2Motif together with its fragment/loss terms and spectra-level loadings (the paper reports 300 Mass2Motifs per data file, so the export should cover the full motif set)
- Motif bubbles are positioned by inter-topic distance and sized by overall prevalence, consistent with the LDAvis design
- Lambda relevance slider correctly re-ranks fragment and neutral-loss terms per motif
- Visual interpretability confirmed by expert review against the SI network subset shown from MS2LDAvis

## Limitations

- Mass2Motif degrees varied from 1 to over 200 spectra; motifs below the ≥10 degree filter were not taken forward for characterization, so rare motifs are under-explored in the visualization workflow.
- Only 30–40 Mass2Motifs per positive-ionization mode file were structurally characterized, so visual inspection of most of the 300 motifs per file remains putative.
- MS2LDA is computationally more costly than simpler tools, and works best only for complex mixtures with information-rich MS2 spectra (e.g., ramped or stepped collision energy).

## Evidence

- [readme] MS2LDAViz web app from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Scope limitation from discussion: [evidence span withheld — anchor status: not_found]
