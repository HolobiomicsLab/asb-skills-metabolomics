---
name: mz-decimal-precision-binning
description: Use when processed matchms Spectrum objects must be turned into Word2Vec
  documents for Spec2Vec training, i.e. after metadata cleaning and peak filtering
  but before training the Word2Vec model with gensim.
when_to_use_negative:
- When the task requires exact fragment-mass matching (e.g. cosine or modified cosine
  score calculations), since binning m/z to 2 decimals discards that precision
- When training a Spec2Vec model on a small dataset — including more m/z decimals
  in the words makes missing-word problems a relevant issue
- When spectra have not yet been processed (metadata cleaned with matchms, peaks restricted
  to m/z [0, 1000], spectra with <10 peaks removed)
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spec2vec
  role: Provides the SpectrumDocument class that converts spectra into documents of
    peak (and loss) words at a chosen decimal precision
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Supplies the processed Spectrum objects (imported from MGF/JSON) that feed
    the document conversion
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:0fa54aa6674a196b55428a88eb5662311a43a5628a4be40312800682f170570f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mz-decimal-precision-binning

## Summary

Converts processed MS/MS spectra into Spec2Vec-ready documents by tokenizing every fragment peak as a word encoding its m/z position rounded to a fixed decimal precision (e.g. 'peak@xxx.xx' at 2 decimals), optionally appending neutral losses (5.0–200.0 Da) as 'loss@xxx.xx' words.

## When to use

Use when processed matchms Spectrum objects must be turned into Word2Vec documents for Spec2Vec training, i.e. after metadata cleaning and peak filtering but before training the Word2Vec model with gensim.

## When NOT to use

- When the task requires exact fragment-mass matching (e.g. cosine or modified cosine score calculations), since binning m/z to 2 decimals discards that precision
- When training a Spec2Vec model on a small dataset — including more m/z decimals in the words makes missing-word problems a relevant issue
- When spectra have not yet been processed (metadata cleaned with matchms, peaks restricted to m/z [0, 1000], spectra with <10 peaks removed)

## Inputs

- Processed/filtered matchms Spectrum objects (cleaned metadata, peaks within m/z [0, 1000], at least 10 peaks per spectrum)
- Neutral-loss window parameters (lower = 5.0 Da, upper = 200.0 Da)
- Decimal-precision setting for m/z words (e.g. 2 decimals)

## Outputs

- Collection of spectrum documents containing 'peak@xxx.xx' words (2-decimal precision) and 'loss@xxx.xx' words
- Serialized document corpus (e.g. spectrum_documents.json) ready for gensim Word2Vec training

## How to apply

Load the cleaned spectra (e.g. the AllPositive deposit from zenodo 10.5281/zenodo.3979010) as matchms Spectrum objects. For each spectrum, tokenize every fragment peak as a word containing its m/z position up to a defined decimal precision — 2 decimals gives 'peak@xxx.xx' — so that nearby m/z values collapse to the same vocabulary word. Compute neutral losses as precursor m/z minus fragment m/z, keep only losses between 5.0 and 200.0 Da, and append them as 'loss@xxx.xx' tokens alongside the peak tokens. Build the documents with spec2vec's SpectrumDocument class and serialize the collection (e.g. to spectrum_documents.json). Rationale: Word2Vec learns from co-occurrences of these words, so the chosen decimal precision defines the effective vocabulary granularity; more decimals inflate vocabulary size and missing-word risk on small training sets.

## Related tools

- **spec2vec** (Provides the SpectrumDocument class that converts spectra into documents of peak (and loss) words at a chosen decimal precision) — https://github.com/iomega/spec2vec
- **matchms** (Supplies the processed Spectrum objects (imported from MGF/JSON) that feed the document conversion) — https://github.com/matchms/matchms

## Examples

```
from spec2vec import SpectrumDocument; docs = [SpectrumDocument(s, n_decimals=2) for s in spectra]
```

## Evaluation signals

- Every generated document contains 'peak@' tokens whose values carry exactly the chosen number of decimal places (e.g. two decimals in 'peak@xxx.xx')
- Every 'loss@' token value falls within the retained 5.0–200.0 Da window
- The serialized corpus (e.g. spectrum_documents.json) exists and parses as valid JSON
- Vocabulary size remains manageable and missing-word fraction stays low for the training set at the chosen decimal precision

## Limitations

- Missing words become a relevant issue when Spec2Vec models are trained on smaller datasets or when more decimals from peak m/z are included in words
- Some peaks and losses not covered by the trained model (about 3% in the study) may not even be chemically meaningful, e.g. many low-Dalton values
- For Spec2Vec to work, the training set must include a large fraction of the features (fragment ions and losses) seen in the query data

## Evidence

- [discussion] After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision: "After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision"
- [discussion] In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as "loss@xxx.xx".: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] missing words will mostly become a relevant issue when Spec2Vec models are trained on smaller datasets, or when more decimals from peak m/z are included in words.: "missing words will mostly become a relevant issue when Spec2Vec models are trained on smaller datasets, or when more decimals from peak m/z are included in words."
- [readme] spec2vec does so for mass fragments and neutral losses in MS/MS spectra.: "spec2vec does so for mass fragments and neutral losses in MS/MS spectra."
- [discussion] we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks.: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
