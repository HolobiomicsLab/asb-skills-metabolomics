---
name: mass-spectra-metadata-cleaning
description: Use when importing a GNPS or similar MS/MS spectral library whose metadata
  and peaks must be cleaned, corrected, and filtered (e.g., ALL_GNPS.json) before
  building training/query subsets.
when_to_use_negative:
- The input is already a cleaned, filtered dataset that meets the ≥10-peak and m/z
  [0, 1000] constraints — re-filtering adds no value.
- The spectra lack any metadata/annotations and no external lookup (e.g., PubChem
  via pubchempy) is possible, so InChIKey-based subsetting cannot be performed.
- The analysis goal requires keeping spectra with fewer than 10 peaks, which this
  pipeline deliberately discards.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Imports the GNPS library, cleans and corrects spectrum metadata, and applies
    the m/z [0, 1000] and <10-peak filters
  repo: https://github.com/matchms/matchms
- name: pubchempy
  role: Runs the automated lookup against PubChem to link spectra to InChIKeys during
    metadata enrichment
- name: spec2vec_gnps_data_analysis
  role: Reference analysis notebooks that apply the cleaning/filtering workflow to
    the GNPS dataset
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:f7923611ee499ccce8569ee419d990b1f699ed1e15a7776d431c98e3e00f7a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pubchempy
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- mass-spectra-metadata-handling
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-spectra-metadata-cleaning

## Summary

A matchms-based procedure to clean and correct MS/MS spectra metadata (e.g., InChIKey annotations) and apply peak filters (m/z range [0, 1000], minimum 10 peaks) to a large GNPS spectral library before downstream similarity modeling or dataset construction.

## When to use

Use when importing a GNPS or similar MS/MS spectral library whose metadata and peaks must be cleaned, corrected, and filtered (e.g., ALL_GNPS.json) before building training/query subsets.

## When NOT to use

- The input is already a cleaned, filtered dataset that meets the ≥10-peak and m/z [0, 1000] constraints — re-filtering adds no value.
- The spectra lack any metadata/annotations and no external lookup (e.g., PubChem via pubchempy) is possible, so InChIKey-based subsetting cannot be performed.
- The analysis goal requires keeping spectra with fewer than 10 peaks, which this pipeline deliberately discards.

## Inputs

- GNPS ALL_GNPS.json spectral library (154,820 positive ionization mode MS/MS spectra)
- MS/MS spectra in formats supported by matchms (mzML, mzXML, msp, MGF, JSON)

## Outputs

- Cleaned, metadata-corrected spectra collection with InChIKey annotations
- AllPositive dataset file (e.g., MGF) with 95,320 spectra (77,092 with InChIKey)
- UniqueInchikey dataset file (e.g., MGF) with 12,797 spectra

## How to apply

Load the positive-ionization library spectra (e.g., the GNPS ALL_GNPS.json library with 154,820 spectra) with matchms, which supports mzML, mzXML, msp, MGF, and JSON formats. Clean and correct the provided metadata with matchms, and run an automated lookup against PubChem (via pubchempy) to link spectra to InChIKeys; in the reference workflow this yielded 128,042 linked spectra and 94,121 spectra with InChIKey annotations after cleaning. Apply peak filters: remove all peaks with m/z outside the range [0, 1000] and discard all spectra retaining fewer than 10 peaks. Split the result by annotation status to derive the AllPositive dataset (95,320 spectra, of which 77,092 carry InChIKeys) and, from the annotated subset, the UniqueInchikey dataset (12,797 spectra with unique InChIKeys). Serialize the resulting collections as dataset files (e.g., MGF) and verify spectrum counts against the expected values before model training.

## Related tools

- **matchms** (Imports the GNPS library, cleans and corrects spectrum metadata, and applies the m/z [0, 1000] and <10-peak filters) — https://github.com/matchms/matchms
- **pubchempy** (Runs the automated lookup against PubChem to link spectra to InChIKeys during metadata enrichment)
- **spec2vec_gnps_data_analysis** (Reference analysis notebooks that apply the cleaning/filtering workflow to the GNPS dataset) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Evaluation signals

- Spectrum counts match the reported values: 95,320 spectra in AllPositive, 77,092 with InChIKey, and 12,797 in UniqueInchikey.
- Every retained spectrum has at least 10 peaks after filtering; no retained peak has m/z outside [0, 1000].
- Metadata cleaning/InChIKey lookup rates are consistent with the reference: 128,042 of 154,820 spectra linked to InChIKeys and 94,121 spectra with InChIKey annotations after cleaning.
- The AllPositive and UniqueInchikey collections serialize to valid dataset files (e.g., MGF) whose counts re-import correctly with matchms.

## Limitations

- Metadata cleaning and InChIKey linkage are lossy: of 154,820 library spectra, only 128,042 could be linked to an InChIKey, and after cleaning only 94,121 retained InChIKey annotations.
- After filtering, some remaining peaks/losses may not be chemically meaningful (e.g., many low-Dalton values).
- Missing-annotation and missing-feature issues become more relevant when working with smaller datasets or more m/z decimals included in derived words.

## Evidence

- [discussion] metadata cleaned with matchms, InChIKey counts: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [discussion] peak filtering thresholds: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
- [intro] resulting dataset sizes after filtering: "After filtering, processing, and removing all spectra with fewer than 10 fragment peaks (see Methods), the remaining AllPositive dataset comprised 95,320 positive ionization mode mass spectra"
- [readme] matchms scope for importing/cleaning spectra: [evidence span withheld — anchor status: external_doc]
- [readme] supported input formats: [evidence span withheld — anchor status: external_doc]
