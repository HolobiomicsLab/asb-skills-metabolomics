---
name: ms2-spectrum-peak-merging
description: Use when multiple ESI-MS2 records of the same compound exist under different
  CID collision energies (e.g. 10–50 V) and a single reference spectrum per compound
  is needed for spectral similarity search.
when_to_use_negative:
- Input spectra are EI-MS or FAB-MS data rather than ESI-MS2 records measured at multiple
  collision energies
- Only a single MS2 scan per compound exists, so there are no multiple CID-energy
  peak lists to merge
- The task requires per-collision-energy spectra rather than one merged spectrum per
  compound
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassBank-cli-tools
  role: Java command line tools (including the validator) used for processing MassBank
    record files
  repo: https://github.com/MassBank/MassBank-cli-tools
- name: MassBank3
  role: Next-generation MassBank web/API software that serves and manages the MassBank-data
    record files from which merged spectra are derived
  repo: https://github.com/MassBank/MassBank3
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:e15907317ada44d3dc3a93bb710781313cdefafcc0c311c2e94be3688ff621fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank-cli-tools
  tools:
  - MassBank-cli-tools
  - MassBank3
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ms2-spectrum-peak-merging

## Summary

Reconstructs MassBank-style artificial merged ESI-MS2 reference spectra by superimposing the peak lists of one compound measured at multiple collision-induced dissociation energies (e.g. 10–50 V), producing a single merged spectrum per compound and instrument class, which improves compound identification precision.

## When to use

Use when multiple ESI-MS2 records of the same compound exist under different CID collision energies (e.g. 10–50 V) and a single reference spectrum per compound is needed for spectral similarity search.

## When NOT to use

- Input spectra are EI-MS or FAB-MS data rather than ESI-MS2 records measured at multiple collision energies
- Only a single MS2 scan per compound exists, so there are no multiple CID-energy peak lists to merge
- The task requires per-collision-energy spectra rather than one merged spectrum per compound

## Inputs

- MassBank ESI-MS2 record files (record format with accession ID and PEAK entries)
- Collision-energy annotations (e.g. 10–50 V) per record
- Precursor/compound identifiers per record

## Outputs

- One artificial merged MS2 spectrum per compound per instrument class
- [Merged QqQ] target set of 856 entries
- [Merged QqTOF] target set of 856 entries
- merged_spectra.csv

## How to apply

Parse MassBank ESI-MS2 record files (MassBank record format with accession ID and PEAK entries), extracting peak lists, precursor/compound identifiers, and collision-energy annotations. Group the parsed records by compound identity and instrument class (QqQ vs QqTOF) so all CID-energy scans of one compound fall into the same group. Superimpose peak lists within each group by merging peaks at matching m/z and summing/integrating their intensities into one artificial merged spectrum per compound. Write one merged record per compound for each instrument class, forming the [Merged QqQ] and [Merged QqTOF] target sets (856 entries each) and save to merged_spectra.csv. The rationale is that merging ESI-MS2 data under different collision-induced dissociation conditions significantly improved identification precision by 21–23% at a similarity score of 0.6, where similarity is computed by a weighted cosine correlation with exponents on peak intensity and m/z optimized for ESI-MS2 data.

## Related tools

- **MassBank-cli-tools** (Java command line tools (including the validator) used for processing MassBank record files) — https://github.com/MassBank/MassBank-cli-tools
- **MassBank3** (Next-generation MassBank web/API software that serves and manages the MassBank-data record files from which merged spectra are derived) — https://github.com/MassBank/MassBank3

## Examples

```
curl http://localhost:8081/MassBank-api/records/count
```

## Evaluation signals

- Output contains exactly 856 merged entries per target set ([Merged QqQ] and [Merged QqTOF])
- Each compound yields exactly one merged spectrum per instrument class
- Merging preserves peak structure: matching m/z peaks across collision energies are combined and intensities summed/integrated, not discarded
- Downstream weighted cosine correlation similarity search (exponents optimized to ESI-MS2 data) reproduces the reported 21–23% precision improvement at a similarity score of 0.6

## Limitations

- Merged spectra are artificial overlays; peak intensities depend on which collision energies were measured, so merging is only meaningful within one instrument class (QqQ vs QqTOF)
- The precision gain of 21–23% is tied to a similarity score threshold of 0.6 with a weighted cosine correlation optimized specifically to ESI-MS2 data
- Custom record data must be in the specified MassBank record format (each file with its own accession ID, peaks, etc.) or the merge step will fail

## Evidence

- [abstract] Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6.: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] weighted cosine correlation optimized to ESI-MS2: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [readme] MassBank record format requirement: [evidence span withheld — anchor status: not_found]
- [readme] MassBank data directory for record files: [evidence span withheld — anchor status: not_found]
