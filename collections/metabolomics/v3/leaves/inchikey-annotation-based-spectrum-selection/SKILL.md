---
name: inchikey-annotation-based-spectrum-selection
description: Use when a cleaned MS/MS library must be split into training spectra
  with structural identifiers (InChIKeys) and query spectra without them, or when
  a deduplicated per-molecule reference set is needed.
when_to_use_negative:
- When the source library spectra have not yet had peaks outside m/z [0, 1000] removed
  or spectra with fewer than 10 peaks discarded — the split should operate on the
  already-filtered AllPositive collection
- When spectra lack reliable InChIKey annotations and no metadata cleaning/lookup
  step (matchms + PubChem via pubchempy) has been run first
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Loads, cleans, and corrects library metadata (InChIKey annotations) and applies
    peak filtering before the annotated/unannotated split
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Downstream consumer of the resulting AllPositive and UniqueInchikey dataset
    files used for model training and benchmarking
  repo: https://github.com/iomega/spec2vec
- name: pubchempy
  role: Automated lookup against PubChem to link library spectra to InChIKeys before
    selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:9ee4730ae688fd0c6a904901d72bf2b54ac59f31ddc163831ff8b4064ca74aa3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Spec2Vec
  - matchms
  - pubchempy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inchikey-annotation-based-spectrum-selection

## Summary

Splits a filtered positive-ionization GNPS spectral library into InChIKey-annotated and unannotated subsets, and further derives a unique-InChIKey subset, to separate training data from query data and avoid molecule overlap.

## When to use

Use when a cleaned MS/MS library must be split into training spectra with structural identifiers (InChIKeys) and query spectra without them, or when a deduplicated per-molecule reference set is needed.

## When NOT to use

- When the source library spectra have not yet had peaks outside m/z [0, 1000] removed or spectra with fewer than 10 peaks discarded — the split should operate on the already-filtered AllPositive collection
- When spectra lack reliable InChIKey annotations and no metadata cleaning/lookup step (matchms + PubChem via pubchempy) has been run first

## Inputs

- Peak-filtered positive ionization mode MS/MS spectra from the GNPS ALL_GNPS.json library (154,820 spectra; m/z restricted to [0, 1000], >=10 peaks retained)
- Spectra with cleaned and corrected metadata including InChIKey annotations (produced via matchms and a PubChem lookup via pubchempy)

## Outputs

- AllPositive dataset file (e.g. MGF format) with 95,320 spectra, of which 77,092 carry InChIKey annotations
- UniqueInchikey dataset file (e.g. MGF format) with 12,797 spectra, all with InChIKey and one spectrum per unique InChIKey
- Held-out query spectra excluded from training to avoid molecule overlap

## How to apply

Start from the m/z-filtered spectra (peaks restricted to m/z [0, 1000]) that retain at least 10 peaks — in the Spec2Vec GNPS workflow this yielded the AllPositive dataset of 95,320 positive ionization mode spectra. Split these into spectra carrying InChIKey annotations (77,092 in the paper) and those without, removing the unannotated spectra from the training pool so there is no molecule overlap between training and query data. From the annotated subset, select only spectra with unique InChIKeys to build the UniqueInchikey dataset (12,797 spectra in the paper). Serialize each subset as a dataset file (e.g. MGF format) and verify spectrum counts against the reported values. Metadata should already have been cleaned with matchms, including an automated PubChem lookup (via pubchempy) that linked 128,042 of 154,820 spectra to an InChIKey before filtering.

## Related tools

- **matchms** (Loads, cleans, and corrects library metadata (InChIKey annotations) and applies peak filtering before the annotated/unannotated split) — https://github.com/matchms/matchms
- **Spec2Vec** (Downstream consumer of the resulting AllPositive and UniqueInchikey dataset files used for model training and benchmarking) — https://github.com/iomega/spec2vec
- **pubchempy** (Automated lookup against PubChem to link library spectra to InChIKeys before selection)

## Examples

```
from matchms.importing import load_from_mgf; spectra = [s for s in load_from_mgf('allpositive.mgf') if s.get('inchikey')]; unique = {s.get('inchikey'): s for s in spectra}
```

## Evaluation signals

- AllPositive dataset contains exactly 95,320 positive ionization mode spectra after removing spectra with fewer than 10 fragment peaks
- Annotated subset within AllPositive contains exactly 77,092 spectra with InChIKey annotations
- UniqueInchikey dataset contains exactly 12,797 spectra, all carrying InChIKeys, with no InChIKey appearing more than once
- No overlap in molecules (InChIKeys) between the training set and the held-out query data
- Serialized dataset files (e.g. MGF) round-trip to the same spectrum counts when reloaded with matchms

## Limitations

- InChIKey coverage is incomplete: only 128,042 of 154,820 ALL_GNPS spectra could be linked to an InChIKey via the PubChem lookup, so many spectra cannot enter the annotated subset
- The split is specific to positive ionization mode data from the GNPS library snapshot (2020-05-11); counts will differ for other snapshots or ionization modes

## Evidence

- [discussion] we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks.: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
- [discussion] This left us with 95,320 spectra (out of which 77,092 with InChIKey) for the AllPositive dataset: "This left us with 95,320 spectra (out of which 77,092 with InChIKey) for the AllPositive dataset"
- [intro] To make sure that there is no overlap regarding molecules between training and query data, we also removed all spectra without InChIKey annotation.: "To make sure that there is no overlap regarding molecules between training and query data, we also removed all spectra without InChIKey annotation."
- [discussion] The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations.: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [discussion] an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]).: "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36])."
