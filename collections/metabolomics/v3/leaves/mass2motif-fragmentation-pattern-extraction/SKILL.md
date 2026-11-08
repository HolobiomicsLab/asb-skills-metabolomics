---
name: mass2motif-fragmentation-pattern-extraction
description: Use when you have preprocessed MS2 spectra (~1,000 MS peaks per sample,
  positive and negative ionization mode) from complex mixtures and want unsupervised
  discovery of conserved fragmentation substructures for de novo annotation.
when_to_use_negative:
- The MS2 spectra are information-poor (e.g., not acquired with ramped or stepped
  collision energy) — MS2LDA works best for complex mixtures where a large number
  of metabolites are fragmented
- The goal is targeted annotation against reference libraries rather than unsupervised
  motif discovery (use spectral matching to Mass Bank/NIST/GNPS instead)
- Only a small spectral dataset is available where per-motif degree would rarely reach
  the >= 10 retention threshold
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3474
tools:
- name: MS2LDA
  role: 'Core framework: represents MS2 spectra as bags of fragments/neutral losses,
    runs LDA inference (including variational Bayes) to learn Mass2Motifs and per-spectrum
    decompositions'
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Visualization of the spectral connectivity / motif network produced from the
    learned motifs
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: Molecular Networking
  role: Complementary spectral network view shown alongside Mass2Motif decompositions
    to relate motifs to molecular families
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:20c628057405ce9106e2495f9749f17d0fa8e2844b354352ba4cd6b8188f4163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  - MS2LDAvis
  - Molecular Networking
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass2motif-fragmentation-pattern-extraction

## Summary

Applies latent Dirichlet allocation topic modeling (MS2LDA) to MS2 fragmentation spectra to discover recurring substructures (Mass2Motifs) as sets of co-occurring fragments and neutral losses, and decomposes each spectrum into these motifs without training data.

## When to use

Use when you have preprocessed MS2 spectra (~1,000 MS peaks per sample, positive and negative ionization mode) from complex mixtures and want unsupervised discovery of conserved fragmentation substructures for de novo annotation.

## When NOT to use

- The MS2 spectra are information-poor (e.g., not acquired with ramped or stepped collision energy) — MS2LDA works best for complex mixtures where a large number of metabolites are fragmented
- The goal is targeted annotation against reference libraries rather than unsupervised motif discovery (use spectral matching to Mass Bank/NIST/GNPS instead)
- Only a small spectral dataset is available where per-motif degree would rarely reach the >= 10 retention threshold

## Inputs

- Preprocessed MS2 spectra corpus (bag of fragments and neutral losses per spectrum), e.g. from beer-extract LC-MS data (~1,000 MS peaks per sample per ionization mode)
- Number of topics parameter (300 Mass2Motifs per data file)
- Mass2LDA preprocessing utilities output (fragments and neutral losses documents)

## Outputs

- motifset.json: 300 discovered Mass2Motifs per data file as fragment/neutral-loss distributions
- motifset_optimized.json
- doc2spec_map.pkl: mapping of documents to original spectra
- convergence_curve.png: LDA training convergence trace
- Per-spectrum Mass2Motif loadings (spectrum decomposition) with motif degrees (number of spectra per motif, 1 to >200)

## How to apply

Load the preprocessed spectra corpus (e.g., the four beer extracts from dx.doi.org/10.5525/gla.researchdata.313, ~1,000 MS peaks per sample per ionization mode) and represent each MS2 spectrum as a bag of fragments and neutral losses using MS2LDA preprocessing utilities. Initialize the LDA model with the number of topics (Mass2Motifs) set to 300 per data file, and train with the implemented inference engines including variational Bayesian inference until the convergence curve stabilizes. Extract each Mass2Motif as a fragment/neutral-loss distribution and compute per-document (MS1 peak / MS2 spectrum) motif loadings. Compute each motif's degree (number of spectra in which it occurs, ranging 1 to over 200) and retain motifs with degree >= 10 for downstream structural characterization. Save motifset.json, motifset_optimized.json, doc2spec_map.pkl, and convergence_curve.png; for beer extracts expect 30-40 structurally characterized motifs per positive-ionization file.

## Related tools

- **MS2LDA** (Core framework: represents MS2 spectra as bags of fragments/neutral losses, runs LDA inference (including variational Bayes) to learn Mass2Motifs and per-spectrum decompositions) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Visualization of the spectral connectivity / motif network produced from the learned motifs) — https://github.com/vdhooftcompmet/MS2LDA
- **Molecular Networking** (Complementary spectral network view shown alongside Mass2Motif decompositions to relate motifs to molecular families)

## Examples

```
pip install ms2lda; git clone https://github.com/vdhooftcompmet/MS2LDA.git
```

## Evaluation signals

- The motif set contains exactly 300 Mass2Motifs per processed data file
- Motif degrees span 1 to over 200 spectra; retained motifs (degree >= 10) yield ~30-40 structurally characterized motifs per positive-ionization file
- The LDA training convergence curve stabilizes before inference is stopped (convergence_curve.png)
- Learned motifs are validated for biochemical relevance by expert review and/or spectral matching against MassBank and NIST/GNPS libraries (19 of the characterized positive-mode beer Mass2Motifs were correctly annotated)
- doc2spec_map.pkl correctly maps every document back to its original MS1 peak / MS2 spectrum

## Limitations

- MS2LDA works best only for complex mixtures with information-rich MS2 spectra (e.g., ramped or stepped collision energy)
- MS2LDA is computationally more costly than simpler tools, but not prohibitively so
- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated motif annotation
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other ionization modes and matrices were not reported

## Evidence

- [discussion] variational Bayes on GNPS: [evidence span withheld — anchor status: not_found]
- [discussion] scope limitation: [evidence span withheld — anchor status: not_found]
