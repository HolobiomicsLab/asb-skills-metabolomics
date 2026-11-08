---
name: motif-count-runtime-benchmarking
description: Use when a run of MS2LDA (or any LDA-style motif decomposition) over
  an MGF spectral corpus must be validated against a published motif count and a training-time
  envelope, e.g. 5,670 GNPS spectra yielding 500 Mass2Motifs.
when_to_use_negative:
- The goal is automated structural annotation of motifs rather than counting them
  - MS2LDA's spectral-matching path requires loss-aware matching that is currently
  unsupported, and the 90%-normalized-score threshold hit only 25%/6% of spectra in
  the original beer/Mass Bank comparison
- The data are not information-rich MS2 spectra from a complex mixture (e.g. DDA with
  a single fixed collision energy) - MS2LDA explicitly works best on ramped or stepped
  collision-energy fragmentation
- Input is already a reduced feature table or a peak list rather than raw MS/MS spectra;
  the skill requires the bag-of-fragments preprocessing step first
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: Trains the LDA model with variational Bayes over the MGF spectral corpus,
    infers Mass2Motifs, and exports motif set, loadings, and model dump for benchmarking.
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDAvis
  role: Visualizes the motif-spectrum network produced by the decomposition, used
    to sanity-check motif occurrence ranges (1 to over 200 spectra).
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:ade69b2b2bb9cbceacc6162ba578f326528d9d02fd90b3cce7a95c62c1d25807
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  - MS2LDAvis
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Motif count and runtime benchmarking of MS2LDA decomposition

## Summary

Benchmark an MS2LDA latent-Dirichlet topic-modeling run by recording how many Mass2Motifs are inferred from a fixed spectral corpus and how long variational Bayes training takes, verifying that the reported decomposition (e.g. 5,670 GNPS spectra into 500 Mass2Motifs) is reproducible.

## When to use

Use when a run of MS2LDA (or any LDA-style motif decomposition) over an MGF spectral corpus must be validated against a published motif count and a training-time envelope, e.g. 5,670 GNPS spectra yielding 500 Mass2Motifs.

## When NOT to use

- The goal is automated structural annotation of motifs rather than counting them - MS2LDA's spectral-matching path requires loss-aware matching that is currently unsupported, and the 90%-normalized-score threshold hit only 25%/6% of spectra in the original beer/Mass Bank comparison
- The data are not information-rich MS2 spectra from a complex mixture (e.g. DDA with a single fixed collision energy) - MS2LDA explicitly works best on ramped or stepped collision-energy fragmentation
- Input is already a reduced feature table or a peak list rather than raw MS/MS spectra; the skill requires the bag-of-fragments preprocessing step first

## Inputs

- MGF-format MS/MS spectral corpus (e.g. 5,670 GNPS spectra from the CSI:FingerID study)
- Mass Bank spectral set (1,953 spectra) for the parallel decomposition
- Configured number of Mass2Motifs (e.g. 500)

## Outputs

- motifset.json with the discovered Mass2Motif set
- Per-spectrum Mass2Motif loadings (which motifs occur in which spectra)
- Binary LDA model dump
- metrics.json recording inferred motif count and elapsed training runtime

## How to apply

Download the target spectral corpus (e.g. the 5,670-spectrum GNPS set from the CSI:FingerID study) in MGF format and preprocess it with MS2LDA so each spectrum becomes a bag of fragments and neutral losses after noise filtering. Run MS2LDA's latent Dirichlet allocation with online variational Bayesian inference, fixing the number of Mass2Motifs to the published value (500 for the GNPS corpus; 300 per beer data file; 500 for the 1,953 Mass Bank spectra). After convergence, count the inferred Mass2Motifs and record the elapsed training runtime. Export the learned motifs to motifset.json, dump the LDA model, and write a metrics.json capturing motif count and runtime so the decomposition can be checked against the expected ~500 motifs for the 5,670-spectrum GNPS input. The count check is the primary invariant; runtime is secondary because MS2LDA is 'more costly than simpler tools, but not prohibitively so'.

## Related tools

- **MS2LDA** (Trains the LDA model with variational Bayes over the MGF spectral corpus, infers Mass2Motifs, and exports motif set, loadings, and model dump for benchmarking.) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDAvis** (Visualizes the motif-spectrum network produced by the decomposition, used to sanity-check motif occurrence ranges (1 to over 200 spectra).)

## Examples

```
ms2lda --input gnps_5670.mgf --num_motifs 500 --output_dir results/ --metrics metrics.json
```

## Evaluation signals

- Inferred Mass2Motif count matches the published value for the corpus: ~500 motifs for 5,670 GNPS spectra; 500 motifs for the 1,953 Mass Bank spectra; 300 motifs per individual beer data file.
- metrics.json exists and contains both motif count and elapsed runtime fields populated with non-zero values.
- Motif degree distribution falls within the reported range: each motif occurs in 1 to over 200 spectra.
- motifset.json loads as valid JSON and the binary LDA model dump deserializes without error.
- Per-spectrum motif loadings sum consistently with the number of motifs assigned to each spectrum (no spectra with motif loadings above the configured motif count).

## Limitations

- MS2LDA is computationally more costly than simpler tools, though the authors state this is 'not prohibitively so' - runtime budgeting should expect longer training than, e.g., MS2Analyzer.
- The decomposition only works well when MS1 peaks are reliably matched to MS2 spectra and when spectra are information-rich (ramped/stepped collision energy); simple-mixture spectra degrade the motif inference.
- Automated motif annotation is loss-aware and thus limited: neutral losses are not currently supported by spectral matching tools, so automated annotation had to omit losses.
- The 19-of-20 validation of automated annotation was done only on characterized positive-mode beer Mass2Motifs at a probability threshold of 0.1; other ionization modes or matrices were not reported.
- No version number for MS2LDA is stated in the paper, so exact version reproducibility cannot be guaranteed from the article alone.

## Evidence

- [full_text] decomposed the GNPS dataset of 5,670 spectra into 500 Mass2Motifs: [evidence span withheld — anchor status: not_found]
