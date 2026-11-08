---
name: fragment-neutral-loss-substructure-analysis
description: Use when raw MS/MS spectra (or preprocessed MS peaks) from a complex
  mixture must be decomposed into shared fragment/neutral-loss substructures for untargeted
  annotation, without relying on prior compound identification.
when_to_use_negative:
- The dataset consists of low-information MS2 spectra (single collision energy, few
  fragments) or simple mixtures — MS2LDA works best on information-rich spectra from
  complex mixtures.
- You only need standard spectral library matching of whole spectra — direct search
  against Mass Bank/NIST/GNPS is faster, though it will miss neutral-loss-driven relationships.
- Compute budget is severely constrained — MS2LDA's variational Bayesian inference
  is more costly than simpler tools (though not prohibitive).
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: Core LDA engine that decomposes MS/MS spectra into Mass2Motifs (fragment +
    neutral-loss topics) and assigns spectra to motifs; used for both discovery and
    fixed-motif inference on external library spectra.
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Produces the molecular network visualization that links spectra sharing Mass2Motifs,
    enabling visual exploration of substructure relationships.
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: 'Contrasted tool: rule-based fragment/neutral-loss extraction, superseded
    here by unsupervised MS2LDA which does not need predefined rules.'
- name: Molecular Networking
  role: GNPS molecular networking output shown alongside MS2LDAvis for comparison
    of motif grouping vs classical spectral similarity networks.
- name: MS-DIAL
  role: Mentioned as a tool that could match fragments to MS1 ions in data-independent
    acquisition data, enabling MS2LDA on DIA datasets.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:8f2e77d5f231a580d86fd40ff2b6140aa8bf27d231130037472f32cf40f4a828
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS-DIAL
  - MS2Analyzer
  - MS2LDA
  - MS2LDAvis
  - Molecular Networking
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Fragment-neutral-loss-substructure-analysis

## Summary

Unsupsupervised MS2LDA topic-modeling decomposition of MS/MS spectra into Mass2Motifs — recurring sets of co-occurring fragments and neutral losses — enabling substructure discovery and de novo annotation of molecules in complex mixtures like beer extracts and urine.

## When to use

Use when raw MS/MS spectra (or preprocessed MS peaks) from a complex mixture must be decomposed into shared fragment/neutral-loss substructures for untargeted annotation, without relying on prior compound identification.

## When NOT to use

- The dataset consists of low-information MS2 spectra (single collision energy, few fragments) or simple mixtures — MS2LDA works best on information-rich spectra from complex mixtures.
- You only need standard spectral library matching of whole spectra — direct search against Mass Bank/NIST/GNPS is faster, though it will miss neutral-loss-driven relationships.
- Compute budget is severely constrained — MS2LDA's variational Bayesian inference is more costly than simpler tools (though not prohibitive).

## Inputs

- Preprocessed MS/MS spectra (mzML or vendor raw from Thermo Q-Exactive Orbitrap / HESI II, converted and peak-picked)
- Mass1 peak lists (~1,000 MS peaks per sample in positive and negative ionization mode)
- GNPS and Mass Bank MS/MS reference spectra (used for fixed-motif inference and validation)
- Optional: pre-characterized Mass2Motif sets from previous MS2LDA runs (e.g. 30 beer motifs)

## Outputs

- Mass2Motif decomposition per dataset (motifset.json / ms2lda_viz.json.gz) with per-spectrum motif membership probabilities
- Structurally characterized Mass2Motifs (fragment and neutral-loss lists) per ionization mode
- Molecular network visualization from MS2LDAvis linking spectra that share motifs
- Annotation counts per spectrum (motif-based vs library spectral matching) and coverage percentages
- Differential expression of motif groups across conditions via PLAGE

## How to apply

Preprocess raw MS/MS runs so each sample yields ~1,000 MS peaks with information-rich MS2 spectra (ramped or stepped collision energy on instruments such as a Thermo Q-Exactive Orbitrap with HESI II). Run LDA (MS2LDA) with ~300 Mass2Motifs per data file, decomposing each spectrum into probabilities over motifs where each motif is a set of fragment masses and neutral losses; validate structurally characterized motifs at a probability threshold of 0.1. For annotation, either submit motif spectra directly to spectral matching, or keep beer-characterized motifs fixed during LDA inference on external libraries (GNPS 5,670 spectra decomposed into 500 motifs, Mass Bank, NIST). Compare motif-annotated coverage (~70% of spectra) against library spectral matching at 90% normalized score (25% Mass Bank, 6% NIST), noting neutral losses must be omitted from library matching because tools do not support them.

## Related tools

- **MS2LDA** (Core LDA engine that decomposes MS/MS spectra into Mass2Motifs (fragment + neutral-loss topics) and assigns spectra to motifs; used for both discovery and fixed-motif inference on external library spectra.) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Produces the molecular network visualization that links spectra sharing Mass2Motifs, enabling visual exploration of substructure relationships.) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Contrasted tool: rule-based fragment/neutral-loss extraction, superseded here by unsupervised MS2LDA which does not need predefined rules.)
- **Molecular Networking** (GNPS molecular networking output shown alongside MS2LDAvis for comparison of motif grouping vs classical spectral similarity networks.)
- **MS-DIAL** (Mentioned as a tool that could match fragments to MS1 ions in data-independent acquisition data, enabling MS2LDA on DIA datasets.)

## Examples

```
pip install ms2lda  # then clone and run: git clone https://github.com/vdhooftcompmet/MS2LDA.git
```

## Evaluation signals

- Around 1,000 MS peaks per sample after preprocessing in both positive and negative ionization mode — fewer suggests poor peak picking.
- 300 Mass2Motifs extracted per data file; structurally characterized motifs should number 30–40 per positive-mode file.
- Per-spectrum motif membership probabilities should be validated at a probability threshold of 0.1; ≥81.5% (positive) and ≥63.3% (negative) of substructure annotations should hold above that cutoff.
- Motif-based annotation yield should be ~3× the library-matched yield: ≥70% of spectra with at least one characterized motif vs 25% Mass Bank / 6% NIST hits at 90% normalized score.
- On the GNPS dataset, decomposition into 500 Mass2Motifs from 5,670 spectra is a reproducible reference point.

## Limitations

- MS2LDA works best only for complex mixtures with information-rich MS2 spectra (e.g. ramped or stepped collision energy); simple matrices or single-energy acquisitions degrade performance.
- MS2LDA relies on reliable matching of MS1 peaks to MS2 spectra; DIA data requires prior fragment-to-MS1 assignment (e.g. via MS-DIAL).
- Neutral losses are not supported by spectral matching tools, so library-based annotation of motifs had to omit losses.
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other ionization modes and matrices were not reported.
- Computationally more costly than simpler tools, though not prohibitively so; no specific version number is stated in the paper.

## Evidence

- [abstract] Our approach extracts biochemically relevant molecular substructures ("Mass2Motifs") from spectra as sets of co-occurring molecular fragments and neutral losses.: "Our approach extracts biochemically relevant molecular substructures ("Mass2Motifs") from spectra as sets of co-occurring molecular fragments and neutral losses."
- [results] we matched spectra to the Mass Bank and National Institute of Standards and Technology (NIST) libraries (SI Appendix, section S2.4) at a threshold of 90% normalized score: "we matched spectra to the Mass Bank and National Institute of Standards and Technology (NIST) libraries (SI Appendix, section S2.4) at a threshold of 90% normalized score"
- [results] Three hundred Mass2Motifs were extracted for each data file and checked for biochemical relevance.: "Three hundred Mass2Motifs were extracted for each data file and checked for biochemical rel- evance."
- [results] the Mass2Motifs characterized from beer were included in the analysis and kept fixed, whereas all other Mass2Motifs are learned during LDA inference: "the Mass2Motifs characterized from beer were included in the analysis and kept fixed, whereas all other Mass2Motifs are learned during LDA inference"
- [discussion] using variational Bayesian inference, the GNPS dataset (5,670 spectra) could be decomposed into 500 Mass2Motifs: [evidence span withheld — anchor status: not_found]
- [discussion] works best for complex mixtures where a large number of metabolites are fragmented and information-rich MS2 spectra are available (e.g., generated by ramped or stepped collision energy): [evidence span withheld — anchor status: not_found]
- [readme] MS2LDA is an advanced tool designed for unsupervised substructure discovery in mass spectrometry data, utilizing topic modeling and providing automated annotation of discovered motifs.: "MS2LDA is an advanced tool designed for unsupervised substructure discovery in mass spectrometry data, utilizing topic modeling and providing automated annotation of discovered motifs."
