---
name: msms-spectrum-document-conversion
description: Use when processed MS/MS spectra (matchms Spectrum objects from MGF/JSON
  such as the AllPositive deposit at zenodo 10.5281/zenodo.3979010) must be tokenized
  into peak and loss word documents to train a Spec2Vec Word2Vec model.
when_to_use_negative:
- You need cosine or modified cosine scores instead — those use raw peaks with relative-intensity
  filtering (<0.01) rather than word documents
- The training dataset is small or uses more m/z decimals, where missing words dominate
  the vocabulary and embeddings degrade
- Spectra have not yet been processed (peaks outside [0, 1000] m/z and spectra with
  <10 peaks not yet removed)
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spec2vec
  role: Provides the SpectrumDocument/SpectrumDocumentWithLosses classes that build
    the peak/loss word documents from matchms spectra
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports and holds the processed spectra (MGF/JSON) that are converted into
    documents; used jointly with spec2vec in the documented pipeline
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:8b471eb0d28f71af9e23567c755356ef86d4626fe6403c05ab982e3c980bee35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# msms-spectrum-document-conversion

## Summary

Converts processed MS/MS spectra into Word2Vec-ready documents in which every fragment peak becomes a 'peak@xxx.xx' word and every neutral loss between 5.0 and 200.0 Da becomes a 'loss@xxx.xx' word, the format required for Spec2Vec model training.

## When to use

Use when processed MS/MS spectra (matchms Spectrum objects from MGF/JSON such as the AllPositive deposit at zenodo 10.5281/zenodo.3979010) must be tokenized into peak and loss word documents to train a Spec2Vec Word2Vec model.

## When NOT to use

- You need cosine or modified cosine scores instead — those use raw peaks with relative-intensity filtering (<0.01) rather than word documents
- The training dataset is small or uses more m/z decimals, where missing words dominate the vocabulary and embeddings degrade
- Spectra have not yet been processed (peaks outside [0, 1000] m/z and spectra with <10 peaks not yet removed)

## Inputs

- Processed/cleaned MS/MS spectra as matchms Spectrum objects (filtered MGF or JSON, e.g. the AllPositive dataset from zenodo 10.5281/zenodo.3979010)

## Outputs

- Collection of spectrum documents containing 'peak@xxx.xx' and 'loss@xxx.xx' tokens (e.g. spectrum_documents.json), ready for gensim Word2Vec training

## How to apply

Load the cleaned spectra into matchms Spectrum objects, then for each spectrum tokenize every fragment peak as a word of the form 'peak@xxx.xx', rounding its m/z position to a fixed 2-decimal precision (2 decimals is the paper's choice; more decimals inflate the missing-word problem). Compute neutral losses as precursor m/z minus fragment m/z and retain only those between 5.0 and 200.0 Da, appending them as 'loss@xxx.xx' tokens alongside the peak tokens. Build the documents with spec2vec's SpectrumDocument/SpectrumDocumentWithLosses classes and serialize the full collection (e.g. to spectrum_documents.json). The rationale is that Word2Vec then learns fragmental relationships from co-occurrences across the corpus of documents, which underpins the Spec2Vec similarity score.

## Related tools

- **spec2vec** (Provides the SpectrumDocument/SpectrumDocumentWithLosses classes that build the peak/loss word documents from matchms spectra) — https://github.com/iomega/spec2vec
- **matchms** (Imports and holds the processed spectra (MGF/JSON) that are converted into documents; used jointly with spec2vec in the documented pipeline) — https://github.com/matchms/matchms

## Examples

```
from spec2vec import SpectrumDocument; documents = [SpectrumDocument(s, n_decimals=2) for s in spectra]
```

## Evaluation signals

- Every document contains 'peak@' tokens whose m/z values carry exactly two decimal places
- Every 'loss@' token value falls within the 5.0–200.0 Da window
- Output file exists and parses as valid JSON (spectrum_documents.json)
- Each document's token set mirrors its source spectrum: one peak word per retained fragment peak plus derived loss words within the window
- Downstream: when training the Word2Vec model, the missing fraction of words absent from the model vocabulary should stay low (paper threshold e.g. <0.05 for accepting similarity scores)

## Limitations

- Spec2Vec needs training data to learn the fragment peak relationships, unlike cosine scores
- Missing words become a relevant issue when models are trained on smaller datasets or when more decimals from peak m/z are included in words
- Some remaining peaks/losses (e.g. many low-Dalton values) might not be chemically meaningful

## Evidence

- [discussion] peak words plus loss words: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] word format with decimal precision: "After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision"
- [discussion] training on documents: "A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37]."
- [discussion] missing words with more decimals: "missing words will mostly become a relevant issue when Spec2Vec models are trained on smaller datasets, or when more decimals from peak m/z are included in words."
- [readme] toolchain roles: "matchms (https://github.com/matchms/matchms) and spec2vec (https://github.com/iomega/spec2vec)"
