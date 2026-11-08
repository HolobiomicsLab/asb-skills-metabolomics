---
name: spectral-peak-to-word-tokenization
description: Use when processed MS/MS spectra (matchms Spectrum objects, e.g. cleaned
  MGF/JSON from the AllPositive dataset) must be turned into corpus documents for
  Spec2Vec/Word2Vec model training.
when_to_use_negative:
- The spectra have not yet been peak-filtered/cleaned (e.g. peaks outside m/z [0,
  1000] removed, spectra with fewer than 10 peaks discarded) — tokenization should
  run after matchms preprocessing
- The goal is cosine or modified cosine scoring, which uses raw peak positions and
  intensities rather than word documents
- The analysis is GC-MS rather than the LC-MS data Spec2Vec was demonstrated on
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: matchms
  role: Import, clean, and hold the spectra as Spectrum objects and provide precursor
    m/z for neutral-loss computation
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: SpectrumDocument / SpectrumDocumentWithLosses classes that build the word-token
    documents from peaks and losses
  repo: https://github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:f69ff25046b2cd6b57dd68a7f7512304040756dcced017813ad49e85d0237e0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-peak-to-word-tokenization

## Summary

Converts processed MS/MS spectra into Word2Vec-style documents by tokenizing each fragment peak as a 'peak@xxx.xx' word (m/z binned to a fixed 2-decimal precision) and appending neutral losses (5.0–200.0 Da) as 'loss@xxx.xx' words, the representation required for Spec2Vec training.

## When to use

Use when processed MS/MS spectra (matchms Spectrum objects, e.g. cleaned MGF/JSON from the AllPositive dataset) must be turned into corpus documents for Spec2Vec/Word2Vec model training.

## When NOT to use

- The spectra have not yet been peak-filtered/cleaned (e.g. peaks outside m/z [0, 1000] removed, spectra with fewer than 10 peaks discarded) — tokenization should run after matchms preprocessing
- The goal is cosine or modified cosine scoring, which uses raw peak positions and intensities rather than word documents
- The analysis is GC-MS rather than the LC-MS data Spec2Vec was demonstrated on

## Inputs

- Processed/filtered MS/MS spectra as matchms Spectrum objects (MGF or JSON, e.g. AllPositive deposit at 10.5281/zenodo.3979010)
- Precursor m/z metadata per spectrum for neutral-loss computation

## Outputs

- Spectrum documents of 'peak@xxx.xx' and 'loss@xxx.xx' word tokens (SpectrumDocument/SpectrumDocumentWithLosses objects)
- Serialized collection file spectrum_documents.json

## How to apply

Load the filtered spectra (e.g. from zenodo 10.5281/zenodo.3979010) into matchms Spectrum objects. Tokenize every fragment peak as a word of the form 'peak@xxx.xx', rounding/binning its m/z position to a fixed 2-decimal precision. Compute neutral losses as precursor m/z minus fragment m/z and retain only those between 5.0 and 200.0 Da, appending them as 'loss@xxx.xx' tokens alongside the peak tokens. Build the documents with spec2vec's SpectrumDocument/SpectrumDocumentWithLosses classes, then serialize the full collection to spectrum_documents.json. Validate that every document contains 'peak@' tokens with exactly two decimal places and every 'loss@' token value falls within the 5.0–200.0 Da window.

## Related tools

- **matchms** (Import, clean, and hold the spectra as Spectrum objects and provide precursor m/z for neutral-loss computation) — https://github.com/matchms/matchms
- **spec2vec** (SpectrumDocument / SpectrumDocumentWithLosses classes that build the word-token documents from peaks and losses) — https://github.com/iomega/spec2vec

## Examples

```
from spec2vec import SpectrumDocument; documents = [SpectrumDocument(s, intensity_weighting_power=0.5) for s in spectra]
```

## Evaluation signals

- spectrum_documents.json exists and is valid JSON
- Every document contains 'peak@' tokens with exactly two decimal places in the m/z position
- Every 'loss@' token value falls within the 5.0–200.0 Da window
- The loss token vocabulary mirrors the paper's 'loss@xxx.xx' format while peak tokens follow 'peak@xxx.xx'
- Spectra with fewer than 10 fragment peaks are absent from the documents (they were removed during filtering)

## Limitations

- Missing words become a relevant issue when Spec2Vec models are trained on smaller datasets, or when more decimals from peak m/z are included in words
- Some remaining peaks/losses not covered by the model may not be chemically meaningful (e.g. many low-Dalton values)
- The training set must include a large fraction of the features (fragment ions and losses) seen in query data for Spec2Vec to work

## Evidence

- [discussion] Tokenization scheme with fixed 2-decimal precision: "After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision"
- [discussion] Neutral loss tokens with 5.0–200.0 Da window: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [readme] Losses and fragments are the learned vocabulary: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
- [discussion] Word2Vec training on the resulting documents: "A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37]."
