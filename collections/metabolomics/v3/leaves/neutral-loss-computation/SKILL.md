---
name: neutral-loss-computation
description: Use when converting processed MS/MS spectra into Spec2Vec/SpectrumDocument
  word tokens and neutral losses must be represented as words alongside peak words
  for Word2Vec model training.
when_to_use_negative:
- When building documents for cosine or modified cosine scoring, which operate on
  raw peaks (with relative intensity >= 0.01) rather than loss word tokens
- When the target Word2Vec model was trained without loss tokens — losses outside
  the learned vocabulary inflate the missing fraction and degrade scores
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spec2vec
  role: Provides the SpectrumDocument/SpectrumDocumentWithLosses classes that assemble
    'loss@xxx.xx' and 'peak@xxx.xx' tokens into documents for Word2Vec training
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Loads and holds the processed spectra (precursor m/z and fragment peaks) from
    which neutral losses are computed
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:5822653f16a6a99e5ff4cd47d70932b403d5229d3af2d3925e22ed193d7035cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# neutral-loss-computation

## Summary

Computes neutral losses (precursor m/z minus fragment m/z) for each MS/MS spectrum and converts them into 'loss@xxx.xx' word tokens that are appended to the 'peak@xxx.xx' peak words when building Word2Vec documents for Spec2Vec training.

## When to use

Use when converting processed MS/MS spectra into Spec2Vec/SpectrumDocument word tokens and neutral losses must be represented as words alongside peak words for Word2Vec model training.

## When NOT to use

- When building documents for cosine or modified cosine scoring, which operate on raw peaks (with relative intensity >= 0.01) rather than loss word tokens
- When the target Word2Vec model was trained without loss tokens — losses outside the learned vocabulary inflate the missing fraction and degrade scores

## Inputs

- Processed matchms Spectrum objects (cleaned MGF/JSON spectra, e.g. from the AllPositive dataset)
- Precursor m/z metadata for each spectrum (required for loss computation)

## Outputs

- Spectrum documents containing 'peak@xxx.xx' tokens plus 'loss@xxx.xx' tokens (only losses in the 5.0-200.0 Da window)
- Serialized document collection (e.g. spectrum_documents.json) ready for gensim Word2Vec training

## How to apply

After loading processed spectra (filtered MGF/JSON, e.g. from the AllPositive deposit at doi:10.5281/zenodo.3979010) into matchms Spectrum objects, compute each neutral loss as the precursor m/z minus the fragment peak m/z. Retain only losses between 5.0 and 200.0 Da, as used in the Spec2Vec paper's document conversion. Format each retained loss as a word token 'loss@xxx.xx' with the m/z position rounded to two decimal places, mirroring the 'peak@xxx.xx' tokens already present in the document. Append the loss tokens alongside the peak tokens when building documents with spec2vec's SpectrumDocument/SpectrumDocumentWithLosses classes, then serialize the collection (e.g. spectrum_documents.json) for Word2Vec training with gensim. Note that Spec2Vec word vectors depend on a large training corpus — losses must be present in the trained model vocabulary, otherwise they count toward the 'missing words' fraction.

## Related tools

- **spec2vec** (Provides the SpectrumDocument/SpectrumDocumentWithLosses classes that assemble 'loss@xxx.xx' and 'peak@xxx.xx' tokens into documents for Word2Vec training) — https://github.com/iomega/spec2vec
- **matchms** (Loads and holds the processed spectra (precursor m/z and fragment peaks) from which neutral losses are computed) — https://github.com/matchms/matchms

## Examples

```
from spec2vec import SpectrumDocument; document = SpectrumDocument(spectrum, n_decimals=2)
```

## Evaluation signals

- Every 'loss@xxx.xx' token value in the output documents falls within the 5.0-200.0 Da window
- All token values (peak and loss) carry exactly two decimal places, matching the defined decimal precision
- Each loss equals (precursor m/z - fragment m/z) for some peak in the source spectrum, within rounding tolerance
- Output file (e.g. spectrum_documents.json) exists, parses as valid JSON, and every document contains at least 'peak@' tokens
- Document counts match the input spectrum count after filtering (e.g. 95,320 AllPositive spectra)

## Limitations

- Neutral losses far outside the trained vocabulary contribute to a high 'missing words' fraction; a threshold (e.g. missing fraction < 0.05) can avoid scoring spectra outside the learned peaks/losses
- Not all remaining peaks/losses may be chemically meaningful (e.g. many low-Dalton values)
- Missing words become more relevant with smaller training datasets or when more m/z decimals are included in words

## Evidence

- [discussion] In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”.: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] Spectra are converted to documents of peak and loss words: "After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision"
- [discussion] missing words issue for losses: "missing words will mostly become a relevant issue when Spec2Vec models are trained on smaller datasets, or when more decimals from peak m/z are included in words."
- [discussion] By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided.: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [readme] spec2vec learns fragment/loss relationships: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
