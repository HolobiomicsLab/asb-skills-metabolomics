---
name: lda-topic-model-inference-on-mass-spectra
description: Use when you have a corpus of unlabelled MS2 spectra (e.g. ~1,000 MS
  peaks per sample per ionization mode from complex mixtures) and want unsupervised
  discovery of conserved fragment/neutral-loss substructures plus per-spectrum motif
  loadings.
when_to_use_negative:
- Spectra come from simple mixtures with information-poor MS2 spectra rather than
  complex mixtures fragmented with ramped or stepped collision energy — the article
  notes MS2LDA works best with a large number of fragmented metabolites
- The goal is direct automated library annotation of spectra including neutral losses
  — losses are not currently supported by spectral matching tools and had to be omitted
- Computation budget is minimal and a simpler targeted/similarity tool would suffice
  — MS2LDA is computationally more costly than simpler tools, though not prohibitively
  so
edam_topics:
- http://edamontology.org/topic_3395
- http://edamontology.org/topic_3172
tools:
- name: MS2LDA
  role: Framework that performs LDA topic modeling over MS2 spectra represented as
    fragment/neutral-loss bags, yielding Mass2Motifs and per-spectrum motif loadings
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:a34694e8169855e494eb4c467d3587ec134faf8bd0396434148fc642e87d6438
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3395
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# lda-topic-model-inference-on-mass-spectra

## Summary

Applies unsupervised Latent Dirichlet Allocation (as implemented in MS2LDA) to MS2 fragmentation spectra, represented as bags of fragments and neutral losses, to learn recurring substructures (Mass2Motifs) and decompose each spectrum into these motifs without training data.

## When to use

Use when you have a corpus of unlabelled MS2 spectra (e.g. ~1,000 MS peaks per sample per ionization mode from complex mixtures) and want unsupervised discovery of conserved fragment/neutral-loss substructures plus per-spectrum motif loadings.

## When NOT to use

- Spectra come from simple mixtures with information-poor MS2 spectra rather than complex mixtures fragmented with ramped or stepped collision energy — the article notes MS2LDA works best with a large number of fragmented metabolites
- The goal is direct automated library annotation of spectra including neutral losses — losses are not currently supported by spectral matching tools and had to be omitted
- Computation budget is minimal and a simpler targeted/similarity tool would suffice — MS2LDA is computationally more costly than simpler tools, though not prohibitively so

## Inputs

- Preprocessed MS2 spectra corpus (bags of fragments and neutral losses), e.g. beer-extract data from dx.doi.org/10.5525/gla.researchdata.313 (~1,000 MS peaks per sample per ionization mode)
- Number of topics parameter (300 Mass2Motifs per data file in the reference workflow)
- Optionally, a set of pre-characterized Mass2Motifs to keep fixed during inference

## Outputs

- motifset.json — discovered Mass2Motifs as fragment/neutral-loss distributions (JSON)
- motifset_optimized.json — optimized motif set
- doc2spec_map.pkl — mapping of documents to original MS2 spectra (pickle)
- convergence_curve.png — LDA training convergence plot
- Per-document motif loadings decomposing each MS2 spectrum into motifs, with motif degree (1 to over 200 spectra)

## How to apply

Load the preprocessed spectra corpus and represent each MS2 spectrum as a bag of fragments and neutral losses using the MS2LDA preprocessing utilities. Initialize the LDA model with the number of topics (Mass2Motifs) set to 300 and train with the implemented inference engines — including variational Bayesian inference — until the training convergence curve stabilizes. Extract each Mass2Motif as a fragment/neutral-loss distribution and compute per-document (MS1 peak / MS2 spectrum) motif loadings that decompose every spectrum into motifs. Compute each motif's degree (number of spectra in which it occurs) and retain motifs with degree >= 10 for downstream structural characterization. Save the outputs as motifset.json, motifset_optimized.json, doc2spec_map.pkl (mapping documents to original spectra), and convergence_curve.png; validated variants fix beer-characterized Mass2Motifs while the remaining motifs are learned during LDA inference on external libraries such as GNPS.

## Related tools

- **MS2LDA** (Framework that performs LDA topic modeling over MS2 spectra represented as fragment/neutral-loss bags, yielding Mass2Motifs and per-spectrum motif loadings) — https://github.com/vdhooftcompmet/MS2LDA

## Examples

```
pip install ms2lda
```

## Evaluation signals

- The motif set contains exactly 300 Mass2Motifs per data file, matching the reference configuration
- The LDA training convergence curve stabilizes (convergence_curve.png) before motif extraction
- Motif degrees span the expected range of 1 to over 200 spectra; motifs with degree < 10 are excluded from downstream structural characterization
- 30–40 Mass2Motifs per positive-ionization-mode file are available for expert structural characterization (biochemical relevance review)
- When decomposing an external corpus (e.g. the 5,670-spectrum GNPS dataset) with fixed known motifs, variational Bayesian inference yields ~500 Mass2Motifs

## Limitations

- MS2LDA works best only for complex mixtures with information-rich MS2 spectra, e.g. generated by ramped or stepped collision energy
- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated Mass2Motif annotation
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs (19 correctly annotated); other ionization modes and matrices were not reported
- MS2LDA relies on reliable matching of MS1 peaks to MS2 spectra

## Evidence
