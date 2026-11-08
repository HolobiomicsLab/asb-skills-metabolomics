---
name: tandem-mass-spectrometry-data-processing
description: Use when multiple ESI-MS2 records of a single compound exist at different
  collision-induced dissociation energies and you must merge them into one reference
  spectrum or score searches with a weighted cosine correlation optimized for ESI-MS2
  data.
when_to_use_negative:
- Input is EI-MS or other single-spectrum data without multiple collision-energy measurements
  — there is nothing to merge across CID conditions
- You need to query MassBank records via the modern REST API/web app (MassBank3) rather
  than perform offline spectrum merging and similarity scoring
- The records are not in the specified MassBank record format (each file must contain
  its own accession ID, peaks, etc.)
edam_operation: http://edamontology.org/operation_3645
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassBank3
  role: Next-generation MassBank web app and REST API providing access to the spectral
    reference library and record data used for merging and search
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-cli-tools
  role: Java command line tools including the validator for MassBank record files
    processed in the merging workflow
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:1129bea38f2a74ec45630345d9b89e2dc17c24425e876deea5d8a4150a617a56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank-cli-tools
  - MassBank3
  edam_operation: http://edamontology.org/operation_3645
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tandem-mass-spectrometry-data-processing

## Summary

Processing of ESI-MS2 records from MassBank by merging spectra of the same compound measured at multiple collision energies (e.g. 10–50 V) into one artificial merged spectrum per compound and instrument class, which improves compound identification precision by 21–23% at a similarity score of 0.6.

## When to use

Use when multiple ESI-MS2 records of a single compound exist at different collision-induced dissociation energies and you must merge them into one reference spectrum or score searches with a weighted cosine correlation optimized for ESI-MS2 data.

## When NOT to use

- Input is EI-MS or other single-spectrum data without multiple collision-energy measurements — there is nothing to merge across CID conditions
- You need to query MassBank records via the modern REST API/web app (MassBank3) rather than perform offline spectrum merging and similarity scoring
- The records are not in the specified MassBank record format (each file must contain its own accession ID, peaks, etc.)

## Inputs

- MassBank ESI-MS2 record files (record format with accession ID, PEAK list, collision-energy annotation, compound identifiers)
- Peak lists of MS2 spectra measured at multiple collision energies (e.g. 10–50 V) per compound
- Instrument class metadata distinguishing QqQ and QqTOF measurements

## Outputs

- One artificially merged MS2 spectrum per compound per instrument class ([Merged QqQ] and [Merged QqTOF] sets, 856 entries each)
- merged_spectra.csv containing the merged spectra
- Similarity search results scored by weighted cosine correlation at a similarity score threshold of 0.6

## How to apply

Parse MassBank ESI-MS2 record files (MassBank record format with accession IDs and PEAK lists) to extract peak lists, precursor/compound identifiers, and collision-energy annotations. Group records by compound identity and instrument class (QqQ vs QqTOF) so all MS2 scans of one compound at CID energies of roughly 10–50 V fall in one group. Superimpose peak lists within each group by merging peaks at matching m/z and combining intensities, producing one artificial merged spectrum per compound per instrument class (the [Merged QqQ] and [Merged QqTOF] target sets of 856 entries each, e.g. saved to merged_spectra.csv). For identification, score queries against references using a weighted cosine correlation with weighting exponents on peak intensity and m/z optimized to ESI-MS2 data; a similarity score of 0.6 is the operating threshold reported to give the 21–23% precision improvement. Validate that merged output preserves peak structure so that downstream similarity search reproduces the reported precision gains.

## Related tools

- **MassBank3** (Next-generation MassBank web app and REST API providing access to the spectral reference library and record data used for merging and search) — https://github.com/MassBank/MassBank3
- **MassBank-cli-tools** (Java command line tools including the validator for MassBank record files processed in the merging workflow) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
python scripts/merge_spectra.py --records data/MassBank-data --group-by compound,instrument_class --output merged_spectra.csv
```

## Evaluation signals

- Merged output contains exactly 856 entries per target set ([Merged QqQ] and [Merged QqTOF])
- Merging preserves peak structure such that weighted-cosine similarity search reproduces the reported 21–23% precision improvement at a similarity score of 0.6
- Each compound yields exactly one merged spectrum per instrument class after grouping records by compound identity and QqQ vs QqTOF class
- Input record files pass MassBank record-format validation (accession ID, peaks, and required fields present; filenames match accession IDs and end with .txt)
- Similarity scores are computed with weighting exponents on peak intensity and m/z optimized to ESI-MS2 data, not a plain unweighted cosine

## Limitations

- Merged spectra are artificial composites — the precision gain (21–23% at similarity score 0.6) was demonstrated for ESI-MS2 data of synthetic drugs/authentic compounds and may not transfer to other data types
- Merging requires records grouped consistently by compound identity and instrument class; mixing QqQ and QqTOF data or mis-parsing collision-energy annotations breaks the grouping
- MassBank is a distributed database (each research group serves its own data), so record availability and content may vary across contributors

## Evidence

- [abstract] For each chemical compound, the ESI-MS2 spectra measured under multiple collision energies (e.g. 10-50 V) are overlaid and merged into a single artificially merged MS2 spectrum: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] Weighted cosine correlation scoring optimized to ESI-MS2: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [readme] MassBank record format requirement for input files: [evidence span withheld — anchor status: not_found]
- [intro] MassBank3 as the maintained software for the reference library: "The new MassBank web app is maintained in repo https://github.com/MassBank/MassBank3."
