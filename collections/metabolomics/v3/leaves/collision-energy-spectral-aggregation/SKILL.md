---
name: collision-energy-spectral-aggregation
description: Use when you have multiple ESI-MS2 record files (MassBank record format
  with accession ID, peaks, collision-energy annotation) of the same compound at different
  CID energies and need one merged reference spectrum per compound.
when_to_use_negative:
- Input spectra are EI-MS or FAB-MS records rather than ESI-MS2 measured under varying
  collision-induced dissociation conditions — merging was designed for ESI-MS2 CID
  series
- Each compound has only a single collision-energy spectrum, so there is nothing to
  overlay and merging cannot add information
- Records of the same compound were acquired on different instrument classes (QqQ
  vs QqTOF) but you need a single instrument-specific library — do not merge across
  instrument classes
tools:
- name: MassBank/MassBank3
  role: Serves and provides access to the merged reference spectra and record data
    (REST API, record file format) used as the basis for merging
  repo: https://github.com/MassBank/MassBank3
- name: MassBank/MassBank-cli-tools
  role: Command line tools including the validator for MassBank record files used
    when parsing records
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:d96f2db13c0561109d3c6f50bcf54663f69017550b35ebe90dc11bf795e0d946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank/MassBank-cli-tools
  - MassBank/MassBank3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# collision-energy-spectral-aggregation

## Summary

Reconstructs MassBank-style artificially merged ESI-MS2 reference spectra by superimposing peak lists of one compound measured across multiple collision-induced dissociation energies (e.g. 10-50 V), producing one merged spectrum per compound per instrument class to boost compound identification precision.

## When to use

Use when you have multiple ESI-MS2 record files (MassBank record format with accession ID, peaks, collision-energy annotation) of the same compound at different CID energies and need one merged reference spectrum per compound.

## When NOT to use

- Input spectra are EI-MS or FAB-MS records rather than ESI-MS2 measured under varying collision-induced dissociation conditions — merging was designed for ESI-MS2 CID series
- Each compound has only a single collision-energy spectrum, so there is nothing to overlay and merging cannot add information
- Records of the same compound were acquired on different instrument classes (QqQ vs QqTOF) but you need a single instrument-specific library — do not merge across instrument classes

## Inputs

- MassBank ESI-MS2 record files in MassBank Record Format (.txt, each with its own accession ID and PEAKS block), including collision-energy annotations such as 10-50 V
- Compound identity (precursor/compound identifiers) and instrument-class labels (QqQ vs QqTOF) per record

## Outputs

- One artificially merged MS2 spectrum per compound per instrument class ([Merged QqQ] and [Merged QqTOF] target sets of 856 entries each)
- merged_spectra.csv with the merged peak lists (matched m/z, integrated intensities)

## How to apply

Parse the MassBank ESI-MS2 record files, extracting peak lists, precursor/compound identifiers, and the collision-energy annotation (e.g. 10-50 V) per record. Group the parsed records by compound identity AND instrument class (QqQ vs QqTOF) so all CID-energy scans of one compound fall in the same group. Superimpose peak lists within each group by merging peaks at matching m/z and summing/integrating their intensities into one artificial merged spectrum per compound. Write one merged record per compound for each instrument class, forming the [Merged QqQ] and [Merged QqTOF] target sets, and save to merged_spectra.csv. The rationale is that merging across collision energies stabilizes the peak pattern so that the weighted cosine correlation similarity search (weighting exponents on peak intensity and m/z optimized to ESI-MS2 data) reproduces the reported 21-23% precision improvement at a similarity score of 0.6.

## Related tools

- **MassBank/MassBank3** (Serves and provides access to the merged reference spectra and record data (REST API, record file format) used as the basis for merging) — https://github.com/MassBank/MassBank3
- **MassBank/MassBank-cli-tools** (Command line tools including the validator for MassBank record files used when parsing records) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
python merge_esims2.py --data MassBank-data --group-by compound,instrument_class --merge-on mz --out merged_spectra.csv
```

## Evaluation signals

- Output merged_spectra.csv contains exactly 856 merged entries per target set for both [Merged QqQ] and [Merged QqTOF]
- Each compound has exactly one merged mass spectrum per instrument class after grouping by compound identity and instrument class
- Merged records preserve peak structure such that downstream similarity search with the ESI-MS2-optimized weighted cosine correlation at similarity score 0.6 yields the reported 21-23% precision improvement
- Every merged record retains its accession ID and peaks in valid MassBank Record Format
- Merged peaks only combine intensities at matching m/z values from records within the same QqQ or QqTOF group

## Limitations

- Requires records to be in the specified MassBank format (own accession ID, peaks, etc.); non-conforming files must be converted or excluded
- Merging is defined per instrument class, so QqQ and QqTOF data of the same compound must not be pooled into one merged spectrum
- The 21-23% precision improvement is tied to a similarity score threshold of 0.6 with weighting exponents optimized for ESI-MS2; results may not transfer to other ionization modes or similarity metrics

## Evidence

- [abstract] Merging precision improvement: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] Weighted cosine correlation similarity: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [readme] MassBank record format requirement: [evidence span withheld — anchor status: not_found]
