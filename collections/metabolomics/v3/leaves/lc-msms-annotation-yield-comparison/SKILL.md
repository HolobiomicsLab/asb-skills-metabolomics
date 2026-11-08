---
name: lc-msms-annotation-yield-comparison
description: Use when comparing annotation coverage of unsupervised motif-based substructure
  discovery (MS2LDA Mass2Motifs) against spectral library matching (Mass Bank/NIST)
  on the same MS/MS spectra.
when_to_use_negative:
- Input is a feature table or already-annotated metabolite list rather than raw/preprocessed
  MS2 spectra
- The dataset is not an information-rich complex mixture (e.g. spectra from single
  collision energy without ramped or stepped collision energy), since MS2LDA works
  best only for such data
- The goal is confident de novo structure elucidation rather than relative annotation-coverage
  benchmarking; motif annotation is putative only
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Decomposes MS/MS spectra into Mass2Motifs and computes motif-membership probabilities
    used to count motif-annotated spectra
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Visualizes the Mass2Motif-spectrum network used to inspect motif annotations
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: Mass Bank
  role: Reference library for the library-matching arm of the comparison at 90% normalized
    score
- name: NIST MS/MS database
  role: Reference library for the library-matching arm of the comparison at 90% normalized
    score
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:6d1c3af8372ef4fa40fd1d7958a0549fd47afc18f32f84c14768c3c04be93797
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  - MS2LDAvis
  - Mass Bank
  - NIST MS/MS database
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# LC-MS/MS annotation yield comparison

## Summary

Benchmarks metabolite annotation coverage achieved by MS2LDA Mass2Motif decomposition against classical spectral library matching (Mass Bank, NIST) on the same LC-MS/MS spectra, quantifying the fold-gain in annotated molecules.

## When to use

Use when comparing annotation coverage of unsupervised motif-based substructure discovery (MS2LDA Mass2Motifs) against spectral library matching (Mass Bank/NIST) on the same MS/MS spectra.

## When NOT to use

- Input is a feature table or already-annotated metabolite list rather than raw/preprocessed MS2 spectra
- The dataset is not an information-rich complex mixture (e.g. spectra from single collision energy without ramped or stepped collision energy), since MS2LDA works best only for such data
- The goal is confident de novo structure elucidation rather than relative annotation-coverage benchmarking; motif annotation is putative only

## Inputs

- Precomputed MS2LDA decomposition files (motifset.json, ms2lda_viz.json.gz) for the LC-MS/MS dataset
- Structurally characterized Mass2Motif set (fragment/neutral-loss patterns), e.g. 30 beer motifs from SI Appendix section S2.1
- Peak MS/MS spectra (pseudo-spectra) per ionization mode (~1,000 MS peaks per beer sample)
- Reference spectral libraries: Mass Bank and NIST MS/MS database

## Outputs

- Per-mode counts/percentages of spectra annotated by Mass2Motif membership vs Mass Bank vs NIST library hits
- Ratio of motif-based to library-based annotation yield (expected ~3x in the beer case)
- Motif-membership assignments per spectrum at probability threshold 0.1

## How to apply

Load the precomputed MS2LDA decomposition of the LC-MS/MS dataset (motifset.json / ms2lda_viz.json.gz) together with the set of structurally characterized Mass2Motifs (30 beer motifs in the reference study, from SI Appendix section S2.1). Mark a molecule as motif-annotated when at least one characterized Mass2Motif (a fragment and neutral-loss pattern) loads on its spectrum above the 0.1 probability threshold used to validate substructure annotations. Match the same spectra as pseudo-spectra against Mass Bank and NIST MS/MS libraries at a 90% normalized score threshold, omitting neutral losses because spectral matching tools do not support them. Tabulate the number of molecules annotated by each strategy per ionization mode and compute the ratio; in the four beer extracts, Mass Bank hits covered 25% and NIST 6% of spectra, whereas at least one characterized Mass2Motif covered an average of 70%, giving the ~3x gain. The comparison can also be run with motif spectra themselves subjected to spectral matching for automated motif annotation.

## Related tools

- **MS2LDA** (Decomposes MS/MS spectra into Mass2Motifs and computes motif-membership probabilities used to count motif-annotated spectra) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Visualizes the Mass2Motif-spectrum network used to inspect motif annotations) — https://github.com/vdhooftcompmet/MS2LDA
- **Mass Bank** (Reference library for the library-matching arm of the comparison at 90% normalized score)
- **NIST MS/MS database** (Reference library for the library-matching arm of the comparison at 90% normalized score)

## Examples

```
git clone https://github.com/vdhooftcompmet/MS2LDA.git && python -m ms2lda --input motifset.json --viz ms2lda_viz.json.gz
```

## Evaluation signals

- Motif-based annotation count is approximately three times the library-matched count (beer case: ~70% vs 25%/6% of spectra)
- Library-matching arm reproduces reported hit rates of 25% (Mass Bank) and 6% (NIST) of spectra at 90% normalized score threshold
- Motif membership calls use the same 0.1 probability threshold at which 81.5% and 63.3% of substructure annotations were validated
- Counts are tabulated separately for positive and negative ionization mode
- Neutral losses are excluded from the spectral-matching arm, matching the stated limitation of library tools

## Limitations

- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated library annotation, which disadvantages the library arm
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs (19 of 30 correctly annotated); other modes/matrices not reported
- MS2LDA works best only for complex mixtures with information-rich MS2 spectra (e.g. ramped or stepped collision energy)
- MS2LDA is computationally more costly than simpler tools, though not prohibitively so

## Evidence

- [discussion] MS2LDA scope limitation to complex mixtures: [evidence span withheld — anchor status: not_found]
