---
name: msms-spectral-dataset-preprocessing
description: Use when raw MS/MS spectra (MGF, mzML, msp, JSON from GNPS or similar
  libraries) must be cleaned and harmonized before training spectral embeddings, computing
  cosine/modified-cosine similarities, or building molecular networks.
when_to_use_negative:
- Input is already a cleaned, feature-annotated spectrum collection that matches the
  downstream model's training distribution — re-filtering with different thresholds
  (e.g., a new minimum-peak count) would break comparability with the published model.
- The task is single-pair spectral comparison of already-curated library entries rather
  than corpus-level preparation — the corpus filters (m/z [0,1000], >=10 peaks) add
  nothing there.
- Data is GC-MS, which the study explicitly did not assess ('In the present work,
  we have only demonstrated performance on LC-MS data').
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Imports, cleans, corrects metadata for, and peak-filters MS/MS spectra prior
    to similarity scoring
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converts filtered spectra into documents of peak/loss words and provides the
    SpectrumDocument workflow
  repo: https://github.com/iomega/spec2vec
- name: PubChem (via pubchempy)
  role: Automated lookup to link spectra to InChIKey annotations during metadata enrichment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:ccce5559626bcf867164cb9f255f14de8535621eff385a5aeae6a941bc67b75a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem (via pubchempy)
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msms-spectral-dataset-preprocessing

## Summary

Clean and filter MS/MS spectra collections with matchms before spectral-similarity modeling or networking: remove peaks outside a defined m/z range, discard low-peak spectra, correct metadata, and optionally derive InChIKey-annotated subsets so downstream models (e.g., Spec2Vec Word2Vec training) operate on a consistent corpus.

## When to use

Use when raw MS/MS spectra (MGF, mzML, msp, JSON from GNPS or similar libraries) must be cleaned and harmonized before training spectral embeddings, computing cosine/modified-cosine similarities, or building molecular networks.

## When NOT to use

- Input is already a cleaned, feature-annotated spectrum collection that matches the downstream model's training distribution — re-filtering with different thresholds (e.g., a new minimum-peak count) would break comparability with the published model.
- The task is single-pair spectral comparison of already-curated library entries rather than corpus-level preparation — the corpus filters (m/z [0,1000], >=10 peaks) add nothing there.
- Data is GC-MS, which the study explicitly did not assess ('In the present work, we have only demonstrated performance on LC-MS data').

## Inputs

- Raw MS/MS spectra file (MGF, mzML, mzXML, msp, or JSON; e.g., GNPS ALL_GNPS.json library)
- Spectral metadata (precursor m/z, ionization mode, parent mass)
- Optional compound structure links (InChIKeys via PubChem/pubchempy lookup)

## Outputs

- Cleaned, filtered spectrum list (e.g., AllPositive dataset of 95,320 positive-ionization spectra)
- InChIKey-annotated subset (e.g., 77,092 spectra with InChIKey; 12,797-spectrum UniqueInchiKeys dataset)
- Spectrum 'documents' of peak and loss words ready for Word2Vec/Spec2Vec training or scoring

## How to apply

Import spectra into matchms, then clean and correct metadata (e.g., linking spectra to InChIKeys via a PubChem lookup with pubchempy, as done for the GNPS ALL_GNPS.json library). Apply peak filters: remove all peaks with m/z outside [0, 1000] and discard all spectra with fewer than 10 fragment peaks, which reduced the raw GNPS-derived corpus to the AllPositive dataset of 95,320 positive-ionization spectra (77,092 with InChIKey). Optionally remove spectra without InChIKey annotation to prevent overlap between training and query data. For cosine/modified-cosine baselines, ignore peaks with relative intensity <0.01 compared to the highest intensity peak; for Spec2Vec, cap the number of kept peaks at 0.5 x parent mass and add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words. Finally convert each filtered spectrum into a document of peak/loss words (a given decimal precision of m/z) before Word2Vec training or similarity scoring.

## Related tools

- **matchms** (Imports, cleans, corrects metadata for, and peak-filters MS/MS spectra prior to similarity scoring) — https://github.com/matchms/matchms
- **spec2vec** (Converts filtered spectra into documents of peak/loss words and provides the SpectrumDocument workflow) — https://github.com/iomega/spec2vec
- **PubChem (via pubchempy)** (Automated lookup to link spectra to InChIKey annotations during metadata enrichment)

## Examples

```
from spec2vec import SpectrumDocument; documents = [SpectrumDocument(spectrum, n_decimals=2) for spectrum in matchms_filtered_spectra]
```

## Evaluation signals

- Spectrum count after filtering falls in the expected ballpark: 95,320 positive-ionization spectra from the GNPS-derived corpus, 77,092 with InChIKey
- No spectrum in the output has fewer than 10 fragment peaks or peaks outside the [0, 1000] m/z range
- All retained peaks/loss words in generated documents are covered by the trained model's vocabulary (missing fraction < ~0.05 threshold if applied)
- Metadata cleaning yields valid InChIKey annotations for the expected fraction (94,121 spectra with InChIKey after matchms cleaning in the paper's pipeline)
- Downstream all-pairs scoring runs without errors (e.g., ~82 million all-pairs Spec2Vec similarities in ~300 s on an Intel i7-8550U)

## Limitations

- Spec2Vec models need the training set to include a large fraction of the fragment/loss features seen in the data; missing words become a real issue for models trained on smaller datasets or when more m/z decimals are included in words
- Filtering thresholds are corpus-specific: removing spectra without InChIKey or with <10 peaks reshapes the dataset and may not generalize to other libraries
- Some remaining peaks/losses not covered by the model may not be chemically meaningful (e.g., many low-Dalton values)

## Evidence

- [discussion] we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks.: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
- [intro] After filtering, processing, and removing all spectra with fewer than 10 fragment peaks (see Methods), the remaining AllPositive dataset comprised 95,320 positive ionization mode mass spectra: "After filtering, processing, and removing all spectra with fewer than 10 fragment peaks (see Methods), the remaining AllPositive dataset comprised 95,320 positive ionization mode mass spectra"
- [discussion] In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”.: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations.: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
