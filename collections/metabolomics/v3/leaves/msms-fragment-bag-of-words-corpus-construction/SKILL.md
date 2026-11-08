---
name: msms-fragment-bag-of-words-corpus-construction
description: Use when raw MS2 spectra (e.g. from beer-extract or other complex-mixture
  LC-MS/MS runs) must be converted into the fragment-and-neutral-loss bag-of-words
  documents that LDA/MS2LDA topic modeling consumes.
when_to_use_negative:
- Input data is already a processed peak/feature table or motif-decomposed spectra
  (no LDA corpus construction needed).
- Only a single simple-mixture MS2 spectrum with sparse fragments is available; MS2LDA
  works best for complex mixtures with information-rich MS2 spectra such as ramped
  or stepped collision energy data, not low-fragmentation single-spectrum input.
- Downstream analysis requires neutral-loss information via spectral-matching tools
  that do not support losses; corpus construction with losses is fine, but if the
  pipeline target is pure spectral matching (no topic modeling), omitting the corpus
  step is more appropriate.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Provides the preprocessing utilities (fragment/neutral-loss extraction, noise
    filtering) and the LDA framework that consumes the constructed bag-of-words corpus.
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDA (latent Dirichlet allocation)
  role: Consumes the bag-of-words corpus as documents for unsupervised Mass2Motif
    topic-model inference.
- name: MS2LDAvis
  role: Visualizes the downstream motif network derived from the constructed corpus,
    aiding interpretation of the learned topics.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:66cabacdab90790e79ce4b558a9b002d855b7b4deb73178e355cf4417ea38b04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  - MS2LDA (latent Dirichlet allocation)
  - MS2LDAvis
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# MS/MS fragment bag-of-words corpus construction

## Summary

Constructs an LDA-ready corpus from MS2 spectra by representing each MS1 peak's fragmentation spectrum as a bag of words built from m/z-binned fragments and derived neutral losses, the preprocessing step that MS2LDA requires before topic-model inference.

## When to use

Use when raw MS2 spectra (e.g. from beer-extract or other complex-mixture LC-MS/MS runs) must be converted into the fragment-and-neutral-loss bag-of-words documents that LDA/MS2LDA topic modeling consumes.

## When NOT to use

- Input data is already a processed peak/feature table or motif-decomposed spectra (no LDA corpus construction needed).
- Only a single simple-mixture MS2 spectrum with sparse fragments is available; MS2LDA works best for complex mixtures with information-rich MS2 spectra such as ramped or stepped collision energy data, not low-fragmentation single-spectrum input.
- Downstream analysis requires neutral-loss information via spectral-matching tools that do not support losses; corpus construction with losses is fine, but if the pipeline target is pure spectral matching (no topic modeling), omitting the corpus step is more appropriate.

## Inputs

- Preprocessed MS2 spectra (fragment lists) from LC-MS/MS runs, e.g. the ~1,000 MS peaks per beer-extract sample in positive and negative ionization mode
- Raw peak tables / spectra from which neutral losses are derived
- MS2LDA preprocessing utilities (Extract neutral losses; Filter out noise)

## Outputs

- Bag-of-words corpus: one document per MS1 peak/MS2 spectrum, with fragment m/z bins and neutral-loss values as the vocabulary
- doc2spec_map.pkl mapping each LDA document back to its original MS1 peak/MS2 spectrum
- Preprocessed corpus ready for LDA inference (e.g. 300-topic beer-extract decomposition)

## How to apply

Preprocess the spectral data so that every MS1 peak's associated MS2 spectrum is represented as a document whose vocabulary consists of binned fragment m/z values and neutral-loss values; the article notes that after preprocessing each beer sample consisted of around 1,000 MS peaks in both positive and negative ionization mode, and that a document should contain fragments and neutral losses so that Mass2Motifs can capture both signals. MS2LDA provides preprocessing utilities (Extract neutral losses; Filter out noise) for building this corpus; run these on the raw peak tables/spectra before invoking LDA. The resulting corpus (one document per MS1 peak/MS2 spectrum) is then handed to the LDA trainer (300 topics for the beer-extract case) and saved artifacts such as doc2spec_map.pkl are produced to link documents back to original spectra. Decision points include the m/z bin width for the fragment vocabulary and whether to retain neutral losses (they are needed for motif interpretability but unsupported by downstream spectral-matching tools, which had to omit them during automated motif annotation).

## Related tools

- **MS2LDA** (Provides the preprocessing utilities (fragment/neutral-loss extraction, noise filtering) and the LDA framework that consumes the constructed bag-of-words corpus.) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDA (latent Dirichlet allocation)** (Consumes the bag-of-words corpus as documents for unsupervised Mass2Motif topic-model inference.)
- **MS2LDAvis** (Visualizes the downstream motif network derived from the constructed corpus, aiding interpretation of the learned topics.)

## Examples

```
pip install ms2lda; python -m ms2lda --preprocess --input raw_spectra.mzML --extract_losses --filter_noise
```

## Evaluation signals

- After preprocessing, the number of documents should match the number of MS1 peaks (~1,000 per beer-extract sample per ionization mode), as stated in the article's results.
- Each document's vocabulary should include both fragment m/z bins and neutral-loss entries; motifs inferred without losses would be degraded for structural characterization.
- The doc2spec_map.pkl mapping must allow every LDA document to be traced back to its original MS1 peak/MS2 spectrum without duplicates or gaps.
- Downstream LDA (e.g. 300-topic beer-extract run) should yield 300 Mass2Motifs per file with motif degrees spanning 1 to over 200, confirming the corpus is correctly formed.
- Noise filtering (Filter out noise step) must not drop information-rich fragments; 30–40 positive-ionization-mode motifs should still be structurally characterized.

## Limitations

- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated annotation despite being included in the corpus.
- MS2LDA works best only for complex mixtures with information-rich MS2 spectra (e.g., ramped or stepped collision energy); sparse spectra from simple mixtures produce weak motifs.
- MS2LDA is computationally more costly than simpler tools, but not prohibitively so.

## Evidence

- [readme] README confirms pip installable CLI: [evidence span withheld — anchor status: external_doc]
