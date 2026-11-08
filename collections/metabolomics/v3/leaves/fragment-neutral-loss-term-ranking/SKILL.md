---
name: fragment-neutral-loss-term-ranking
description: Use when inspecting inferred Mass2Motifs (topic terms are MS2 fragments
  and neutral losses) to decide which m/z fragment/loss terms characterize a motif,
  or when visualizing inter-topic structure at a given degree filter.
when_to_use_negative:
- The task is automated motif annotation by spectral matching (neutral losses are
  not supported by spectral matching tools and had to be omitted) — use MS2LDA's spectral-matching
  route instead
- Data are simple mixtures with few, information-poor MS2 spectra; the paper notes
  MS2LDA works best for complex mixtures with ramped or stepped collision energy
- Motifs below the degree-10 threshold are of interest (rare motifs occurring in fewer
  than 10 spectra) since they are excluded from this inspection filter
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDAvis
  role: 'Customized LDAvis module used to interactively explore LDA output: 2D motif
    projection, prevalence-sized bubbles, and lambda-slider ranking of fragment/neutral-loss
    terms'
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDA
  role: Provides the Visualisation module (MS2LDA.Visualisation) that computes topic-term
    distributions and spectra-level topic prevalences from the trained LDA model,
    and the MS2LDAViz web app that consumes the export
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: LDAvis
  role: Original topic-model visualization approach (Sievert & Shirley 2014) adapted
    by MS2LDAvis to the MS/MS domain with a relevance (lambda) slider for term ranking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:8d481ff250484ae6d44354feac067b6c9b759eb3984ccea1b0314c88230b8ade
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fragment-neutral-loss-term-ranking

## Summary

Ranks the molecular fragment and neutral-loss terms of inferred Mass2Motifs by their relevance using the LDAvis lambda (relevance) slider within MS2LDAvis, a customization of LDAvis for MS/MS topic models. This lets an analyst characterize motifs as biochemically meaningful substructures rather than reading raw topic-term distributions.

## When to use

Use when inspecting inferred Mass2Motifs (topic terms are MS2 fragments and neutral losses) to decide which m/z fragment/loss terms characterize a motif, or when visualizing inter-topic structure at a given degree filter.

## When NOT to use

- The task is automated motif annotation by spectral matching (neutral losses are not supported by spectral matching tools and had to be omitted) — use MS2LDA's spectral-matching route instead
- Data are simple mixtures with few, information-poor MS2 spectra; the paper notes MS2LDA works best for complex mixtures with ramped or stepped collision energy
- Motifs below the degree-10 threshold are of interest (rare motifs occurring in fewer than 10 spectra) since they are excluded from this inspection filter

## Inputs

- Trained LDA model binary (ms2lda.bin) from MS2LDA variational Bayesian inference
- Motif set file (motifset.json) with Mass2Motif fragment and neutral-loss term distributions and document-to-spectra mapping
- Mass2Motif degree values (number of spectra in which each motif occurs)

## Outputs

- Interactive 2D projection of Mass2Motifs as prevalence-sized bubbles (LDAvis-style PCA of inter-topic distance)
- Per-motif ranked lists of fragment and neutral-loss terms at the chosen lambda relevance value
- Compressed JSON visualization export (ms2lda_viz.json.gz) consumable by the MS2LDAViz web app

## How to apply

Load the trained LDA model (ms2lda.bin) and motif set (motifset.json) with fragment and neutral-loss term distributions and the document-to-spectra mapping. Use the MS2LDA Visualisation module to compute topic-term distributions and spectra-level topic prevalences, then render Mass2Motifs as bubbles in 2D by inter-topic distance (bubble area = overall motif prevalence) as in LDAvis (Sievert & Shirley 2014). Interactively move the relevance (lambda) slider to re-rank each motif's fragment and neutral-loss terms, balancing term probability within the motif against term exclusivity to the motif. Restrict inspection to motifs meeting the degree filter (>=10, i.e., present in 10 or more spectra) before structural characterization, then bundle the rendering into the compressed JSON consumed by the MS2LDAViz web app. Validate that ms2lda_viz.json.gz is valid gzipped JSON covering every inferred Mass2Motif (the paper reports 300 Mass2Motifs per data file) with its fragment/loss terms and spectra-level loadings.

## Related tools

- **MS2LDAvis** (Customized LDAvis module used to interactively explore LDA output: 2D motif projection, prevalence-sized bubbles, and lambda-slider ranking of fragment/neutral-loss terms) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDA** (Provides the Visualisation module (MS2LDA.Visualisation) that computes topic-term distributions and spectra-level topic prevalences from the trained LDA model, and the MS2LDAViz web app that consumes the export) — https://github.com/vdhooftcompmet/MS2LDA
- **LDAvis** (Original topic-model visualization approach (Sievert & Shirley 2014) adapted by MS2LDAvis to the MS/MS domain with a relevance (lambda) slider for term ranking)

## Examples

```
git clone https://github.com/vdhooftcompmet/MS2LDA.git; pip install ms2lda  # then launch MS2LDAViz to explore motif term rankings
```

## Evaluation signals

- ms2lda_viz.json.gz exists, is valid gzipped JSON, and parses without error
- The export covers the full inferred motif set (the paper reports 300 Mass2Motifs per data file), each with its fragment/loss terms and spectra-level loadings
- Bubble areas in the 2D projection are proportional to overall motif prevalence and inter-topic distances are meaningful (motifs sharing fragments/losses plot close together)
- Terms re-rank sensibly when the lambda slider moves: at high lambda terms are ranked by in-motif probability, at low lambda by exclusivity to the motif
- Motifs under inspection satisfy the degree filter (present in >= 10 spectra) and expert review against the SI network subset from MS2LDAvis confirms interpretability

## Limitations

- Only motifs with degree >= 10 (present in 10 or more spectra) are inspected at this stage, so rare motifs are not visualized/characterized
- Neutral losses cannot be used in downstream automated spectral matching and had to be omitted from annotation
- Interpretation is qualitative: motifs still require manual structural characterization (the paper structurally characterized only 30–40 Mass2Motifs per positive-ionization file out of 300 extracted)

## Evidence

- [readme] MS2LDAViz app in README: [evidence span withheld — anchor status: external_doc]
