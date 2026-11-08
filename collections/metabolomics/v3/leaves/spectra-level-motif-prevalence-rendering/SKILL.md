---
name: spectra-level-motif-prevalence-rendering
description: Use when a trained MS2LDA model (ms2lda.bin) with an inferred Mass2Motif
  set (motifset.json) must be made explorable interactively, e.g. to pick biochemically
  relevant motifs before further characterization.
when_to_use_negative:
- The analysis goal is peak/fragment annotation of already identified compounds —
  spectral matching against Mass Bank/NIST is the applicable step, not motif prevalence
  visualization.
- Input is a single identified spectrum rather than a corpus of spectra from MS2LDA
  inference; prevalence rendering is meaningless for a one-document model.
- The workflow targets automatic annotation output — note neutral losses are not supported
  by current spectral matching tools and had to be omitted, so they must not be carried
  into the annotation export.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MS2LDA
  role: Source of the trained LDA output (ms2lda.bin, motifset.json) whose Mass2Motifs
    and spectra-level assignments are rendered; its Visualisation module computes
    topic-term distributions and spectra-level topic prevalences
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: 'Customization of LDAvis that adapts the topic-model visualization to MS/MS:
    2D inter-topic projection, prevalence bubbles, and lambda slider for fragment/neutral-loss
    terms'
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAViz
  role: Web application that consumes the exported ms2lda_viz.json.gz for interactive
    exploration and analysis of motifs
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:e227fa1acd14529df67c3a4e13d264283fccd167d202ce751afcde1b85ed1885
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectra-level-motif-prevalence-rendering

## Summary

Renders the LDA inference output of MS2LDA into an interactive MS2LDAvis (customized LDAvis) visualization in which each Mass2Motif is a bubble whose area reflects its overall prevalence across spectra, letting analysts inspect motif–term distributions and spectra-level motif loadings.

## When to use

Use when a trained MS2LDA model (ms2lda.bin) with an inferred Mass2Motif set (motifset.json) must be made explorable interactively, e.g. to pick biochemically relevant motifs before further characterization.

## When NOT to use

- The analysis goal is peak/fragment annotation of already identified compounds — spectral matching against Mass Bank/NIST is the applicable step, not motif prevalence visualization.
- Input is a single identified spectrum rather than a corpus of spectra from MS2LDA inference; prevalence rendering is meaningless for a one-document model.
- The workflow targets automatic annotation output — note neutral losses are not supported by current spectral matching tools and had to be omitted, so they must not be carried into the annotation export.

## Inputs

- ms2lda.bin — trained LDA model binary from MS2LDA inference
- motifset.json — discovered Mass2Motif set with fragment and neutral-loss term distributions
- document-to-spectra mapping linking LDA documents to MS2 spectra

## Outputs

- ms2lda_viz.json.gz — compressed JSON for the MS2LDAViz web app containing every inferred Mass2Motif with its fragment/loss terms and spectra-level loadings
- Interactive MS2LDAvis bubble plot of Mass2Motifs in 2D inter-topic space, bubble area = overall motif prevalence
- Per-motif ranked list of fragment and neutral-loss terms controlled by the lambda relevance slider

## How to apply

Load the trained LDA model binary (ms2lda.bin) together with the motif set JSON (motifset.json) containing fragment/neutral-loss term distributions and document-to-spectra mapping. Use the MS2LDA.Visualisation module to compute topic-term distributions and spectra-level topic prevalences from the LDA output. Adapt the LDAvis approach (Sievert & Shirley 2014) to the MS/MS domain: project Mass2Motifs into 2D by inter-topic distance, draw each motif as a bubble whose area reflects its overall prevalence, and add a relevance (lambda) slider for ranking fragment and neutral-loss terms per motif. Inspect Mass2Motifs at a degree filter of ≥10 (motifs present in 10 or more spectra) before structurally characterizing them. Export the interactive rendering as gzipped JSON (ms2lda_viz.json.gz) consumable by the MS2LDAViz web app, then confirm expert interpretability against the SI network subset produced by MS2LDAvis.

## Related tools

- **MS2LDA** (Source of the trained LDA output (ms2lda.bin, motifset.json) whose Mass2Motifs and spectra-level assignments are rendered; its Visualisation module computes topic-term distributions and spectra-level topic prevalences) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Customization of LDAvis that adapts the topic-model visualization to MS/MS: 2D inter-topic projection, prevalence bubbles, and lambda slider for fragment/neutral-loss terms) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAViz** (Web application that consumes the exported ms2lda_viz.json.gz for interactive exploration and analysis of motifs) — https://github.com/vdhooftcompmet/MS2LDA

## Examples

```
MS2LDA.Visualisation.load_model_and_visualize(ms2lda.bin, motifset.json, min_degree=10)  # produces ms2lda_viz.json.gz for MS2LDAViz
```

## Evaluation signals

- ms2lda_viz.json.gz exists, is valid gzipped JSON, and parses without error
- The export covers the full inferred motif set (the paper reports 300 Mass2Motifs per data file), so every motif must be present with its fragment/loss terms and spectra-level loadings
- Each motif bubble's area matches the motif's overall prevalence computed from the spectra-level topic proportions
- Motifs retained for further characterization satisfy the ≥10-spectra degree filter used in the paper
- Visual interpretability confirmed by expert review against the SI network subset shown from MS2LDAvis

## Limitations

- The prevalence computation relies on reliable matching of MS1 peaks to MS2 spectra; unreliable linkage degrades spectra-level motif loadings.
- Mass2Motif degree varies from 1 to over 200 spectra, so low-prevalence motifs may render as tiny bubbles and escape visual inspection.
- Only a fraction of available fragmentation information is traditionally utilized, and MS2LDA works best for complex mixtures with information-rich MS2 spectra (e.g. ramped or stepped collision energy).

## Evidence

- [results] 300 Mass2Motifs extracted: [evidence span withheld — anchor status: not_found]
- [readme] MS2LDAViz app: [evidence span withheld — anchor status: external_doc]
- [discussion] MS1-to-MS2 reliability caveat: [evidence span withheld — anchor status: not_found]
