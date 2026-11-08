---
name: bag-of-fragments-spectral-representation
description: Use when raw MS2 peak lists (e.g., GNPS MGF spectra from Orbitrap acquisitions
  such as the Q-Exactive) must be converted into fragment/neutral-loss count vectors
  before topic modeling or other unsupervised substructure discovery.
when_to_use_negative:
- Input is already a feature table or motif-loading matrix rather than raw MS2 peak
  lists
- Spectra are information-poor (few fragments, e.g., single collision energy on simple
  mixtures) — MS2LDA works best on complex mixtures with information-rich MS2 spectra
  from ramped or stepped collision energy
- The goal is classical spectral matching against libraries like Mass Bank or NIST,
  where neutral losses are not supported and must be omitted
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Provides the preprocessing module that converts spectra into bag-of-fragments
    representations (fragment extraction, neutral-loss extraction, noise filtering)
    prior to LDA inference
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: Contrasted rule-based substructure search tool; unlike MS2Analyzer, the bag-of-fragments
    + LDA approach is unsupervised
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:8f2c6c0751492693891d000982566daf1a16b8412bbe1c676baf52af76a3e047
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2Analyzer
  - MS2LDA
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bag-of-fragments-spectral-representation

## Summary

Represents each MS/MS spectrum as a bag of co-occurring molecular fragments and neutral losses so that topic models such as LDA (MS2LDA) can decompose spectra into Mass2Motifs. This bag-of-fragments conversion is the preprocessing step that makes spectral data amenable to unsupervised substructure discovery.

## When to use

Use when raw MS2 peak lists (e.g., GNPS MGF spectra from Orbitrap acquisitions such as the Q-Exactive) must be converted into fragment/neutral-loss count vectors before topic modeling or other unsupervised substructure discovery.

## When NOT to use

- Input is already a feature table or motif-loading matrix rather than raw MS2 peak lists
- Spectra are information-poor (few fragments, e.g., single collision energy on simple mixtures) — MS2LDA works best on complex mixtures with information-rich MS2 spectra from ramped or stepped collision energy
- The goal is classical spectral matching against libraries like Mass Bank or NIST, where neutral losses are not supported and must be omitted

## Inputs

- MS/MS spectra in MGF format (e.g., the 5,670-spectrum public GNPS dataset from the CSI:FingerID study)
- Per-spectrum peak lists with precursor masses (to compute neutral losses)
- Raw MS2 data from instruments such as the Thermo Scientific Q-Exactive Orbitrap (mzML/mzXML peak lists after centroiding)

## Outputs

- Bag-of-fragments representation of each spectrum (fragment and neutral-loss count vectors)
- Noise-filtered, preprocessing-ready documents for LDA inference
- Per-spectrum motif loadings and Mass2Motif sets after MS2LDA training (exportable as motifset.json)

## How to apply

Convert each spectrum into a bag-of-fragments representation via the MS2LDA preprocessing module: extract molecular fragments from the MS2 peak list, derive neutral losses (e.g., precursor minus fragment masses), and filter out noise peaks. The resulting bag-of-fragments documents feed directly into latent Dirichlet allocation with variational Bayesian (online variational) inference, as used to decompose the 5,670-spectrum GNPS dataset from the CSI:FingerID study into 500 Mass2Motifs. Keep fragment and loss features paired so motifs emerge as sets of co-occurring fragments and losses; note that neutral losses are not supported by downstream spectral-matching tools and had to be omitted for automated annotation.

## Related tools

- **MS2LDA** (Provides the preprocessing module that converts spectra into bag-of-fragments representations (fragment extraction, neutral-loss extraction, noise filtering) prior to LDA inference) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Contrasted rule-based substructure search tool; unlike MS2Analyzer, the bag-of-fragments + LDA approach is unsupervised)

## Examples

```
git clone https://github.com/vdhooftcompmet/MS2LDA.git; python -m ms2lda.preprocessing --input gnps_5670.mgf --extract-neutral-losses --filter-noise
```

## Evaluation signals

- Every input spectrum yields a non-empty bag of fragments and neutral losses after noise filtering (no dropped spectra without cause)
- Running the full MS2LDA pipeline on the 5,670 GNPS spectra with 500 topics reproduces the reported decomposition into approximately 500 Mass2Motifs (record motif count and runtime in metrics.json)
- Motif degrees (number of spectra in which each motif occurs) fall in the plausible range observed in the paper, from 1 to over 200 spectra
- Neutral losses are present in the representation and flagged for omission before any downstream spectral matching, since matching tools do not support losses

## Limitations

- MS2LDA works best for complex mixtures with information-rich MS2 spectra (e.g., ramped or stepped collision energy); simple spectra yield few useful motifs
- Neutral losses in the bag-of-fragments are not supported by spectral matching tools and had to be omitted in automated annotation
- MS2LDA is computationally more costly than simpler tools, though not prohibitively so
- MS2LDA relies on reliable matching of MS1 peaks to MS2 spectra

## Evidence

- [abstract] Our approach extracts biochemically relevant molecular substructures ("Mass2Motifs") from spectra as sets of co-occurring molecular fragments and neutral losses.: "Our approach extracts biochemically relevant molecular substructures ("Mass2Motifs") from spectra as sets of co-occurring molecular fragments and neutral losses."
- [results] Bag-of-fragments preprocessing in the task workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Neutral losses unsupported by matching tools: "despite the fact that losses are not currently supported by spectral matching tools and had to be omitted"
- [discussion] Best-case data regime: [evidence span withheld — anchor status: not_found]
- [results] Motif degree range observed: "The degree of Mass2Motifs (the number of spectra in which they occurred) varied from 1 to over 200"
