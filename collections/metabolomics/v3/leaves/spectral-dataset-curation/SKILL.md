---
name: spectral-dataset-curation
description: Use when building a cleaned training/reference spectral dataset from
  a raw GNPS library dump and you must reproduce reproducible filter thresholds (m/z
  range, minimum peak count) and InChIKey-annotated subsets.
when_to_use_negative:
- The spectra are already filtered/annotated and the goal is similarity scoring or
  library matching — apply Spec2Vec or cosine scores directly instead of re-curation.
- You need to assign InChIKeys to unannotated spectra rather than assemble filtered
  subsets — use the PubChem automated lookup (pubchempy) step, not the filtering pipeline.
- Working with raw LC-MS runs or GC-MS data rather than an existing compiled spectral
  library — this skill curation path is specific to GNPS library collections.
tools:
- name: matchms
  role: Imports the GNPS library, cleans/corrects metadata, and applies the m/z-range
    and minimum-peak-count filters that define AllPositive and UniqueInchikey subsets
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Downstream consumer of the curated AllPositive and UniqueInchikey datasets
    for training spectral embedding models
  repo: https://github.com/iomega/spec2vec
- name: spec2vec_gnps_data_analysis
  role: Analysis notebooks reproducing the dataset creation and benchmarking workflow
    over the GNPS dataset
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:c29a52f30505a60746aaacfcf579539f7b2755c266ab953f63465d2d82a7e1af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Spec2Vec
  - matchms
  - spec2vec_gnps_data_analysis
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-dataset-curation

## Summary

Curate a large GNPS positive-ionization MS/MS library (ALL_GNPS.json, 154,820 spectra) into training-ready subsets using matchms filtering: peak m/z restriction to [0, 1000], a minimum of 10 peaks per spectrum, InChIKey-based subsetting, and deduplication to unique InChIKeys.

## When to use

Use when building a cleaned training/reference spectral dataset from a raw GNPS library dump and you must reproduce reproducible filter thresholds (m/z range, minimum peak count) and InChIKey-annotated subsets.

## When NOT to use

- The spectra are already filtered/annotated and the goal is similarity scoring or library matching — apply Spec2Vec or cosine scores directly instead of re-curation.
- You need to assign InChIKeys to unannotated spectra rather than assemble filtered subsets — use the PubChem automated lookup (pubchempy) step, not the filtering pipeline.
- Working with raw LC-MS runs or GC-MS data rather than an existing compiled spectral library — this skill curation path is specific to GNPS library collections.

## Inputs

- GNPS ALL_GNPS.json positive-ionization library (154,820 spectra; https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json)
- Per-spectrum metadata including peak lists (m/z, intensities) and InChIKey annotations
- PubChem lookup results (via pubchempy) used to link spectra to InChIKeys (128,042 of 154,820 spectra)

## Outputs

- AllPositive dataset file: 95,320 filtered positive-mode spectra (MGF/JSON)
- UniqueInchikey dataset file: 12,797 spectra with unique InChIKeys
- Annotated subset inventory: 77,092 spectra with InChIKey annotations

## How to apply

Load the InChIKey-resolved positive-ionization spectra from the GNPS ALL_GNPS.json library (154,820 spectra from 2020-05-11) with matchms, first cleaning and correcting metadata. Apply two peak/spectrum filters: remove all peaks with m/z outside the range [0, 1000], then discard all spectra retaining fewer than 10 peaks — this yields the AllPositive dataset of 95,320 spectra. Split out spectra carrying InChIKey annotations (77,092 spectra after metadata cleaning via matchms), and additionally remove spectra without InChIKey annotation to avoid overlap between training and query data. From the annotated subset, select spectra with unique InChIKeys to construct the UniqueInchikey dataset of 12,797 spectra, then serialize the two collections as dataset files (e.g. MGF format) and verify counts against the published values.

## Related tools

- **matchms** (Imports the GNPS library, cleans/corrects metadata, and applies the m/z-range and minimum-peak-count filters that define AllPositive and UniqueInchikey subsets) — https://github.com/matchms/matchms
- **Spec2Vec** (Downstream consumer of the curated AllPositive and UniqueInchikey datasets for training spectral embedding models) — https://github.com/iomega/spec2vec
- **spec2vec_gnps_data_analysis** (Analysis notebooks reproducing the dataset creation and benchmarking workflow over the GNPS dataset) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
from matchms.importing import load_from_json; spectra = load_from_json('ALL_GNPS.json')
```

## Evaluation signals

- Filtered AllPositive dataset contains exactly 95,320 spectra after m/z [0, 1000] restriction and the <10-peak removal.
- Of those, exactly 77,092 spectra carry InChIKey annotations.
- UniqueInchikey dataset contains exactly 12,797 spectra, all with unique InChIKey values.
- No spectrum in the output retains a peak with m/z outside [0, 1000] or fewer than 10 peaks.
- Serialized dataset files (e.g. MGF) round-trip with spectrum counts matching the in-memory filtered collections.

## Limitations

- Metadata cleaning yields 94,121 InChIKey-annotated spectra; spectra lacking annotations must be excluded to avoid molecule overlap between training and query data, shrinking the usable set.
- The curation is tied to a specific library snapshot (GNPS ALL_GNPS.json, 2020-05-11); counts will differ for newer library versions.
- Curated subsets underpin Spec2Vec training, which requires that the training set includes a large fraction of the fragment/loss features seen in query data; smaller curated sets make missing words a relevant issue.

## Evidence

- [discussion] we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
- [discussion] This left us with 95,320 spectra (out of which 77,092 with InChIKey) for the AllPositive dataset: "This left us with 95,320 spectra (out of which 77,092 with InChIKey) for the AllPositive dataset"
- [intro] the remaining AllPositive dataset comprised 95,320 positive ionization mode mass spectra: "the remaining AllPositive dataset comprised 95,320 positive ionization mode mass spectra"
- [discussion] The provided metadata was cleaned and corrected using matchms: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [readme] Original data was obtained from GNPS: [evidence span withheld — anchor status: external_doc]
