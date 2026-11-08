---
name: mgf-spectrum-parsing
description: Use when input is an MGF-formatted MS/MS spectrum file (e.g. pesticides.mgf
  or all_your_spectrums.mgf) and the goal is to load it into matchms Spectrum objects
  for downstream filtering or scoring.
when_to_use_negative:
- Input is already a Spectrum object or an in-memory list of spectra — import/parsing
  is not needed again
- File is mzML, mzXML, msp, JSON, or a metabolomics-USI rather than MGF; use the corresponding
  matchms importer for that format instead
- Metadata fields need cleaning/correction (e.g. harmonizing ionmode or checking InChI/SMILES)
  — that belongs to the matchms filtering pipeline, not to load_from_mgf
edam_operation: http://edamontology.org/operation_0223
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: provides matchms.importing.load_from_mgf, the function used to parse MGF files
    into Spectrum objects
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: underlying layer that matchms's MGF importer is built on top of for file parsing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:90387c9c4705346ad2776ad8c042ce6b64fca88fab8a4388b6391b0526ab5d96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_0223
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mgf-spectrum-parsing

## Summary

Parse NIST-formatted MGF files into matchms Spectrum objects using matchms.importing.load_from_mgf, converting raw MGF peak/metadata blocks into iterable, typed in-memory spectra ready for filtering or scoring.

## When to use

Use when input is an MGF-formatted MS/MS spectrum file (e.g. pesticides.mgf or all_your_spectrums.mgf) and the goal is to load it into matchms Spectrum objects for downstream filtering or scoring.

## When NOT to use

- Input is already a Spectrum object or an in-memory list of spectra — import/parsing is not needed again
- File is mzML, mzXML, msp, JSON, or a metabolomics-USI rather than MGF; use the corresponding matchms importer for that format instead
- Metadata fields need cleaning/correction (e.g. harmonizing ionmode or checking InChI/SMILES) — that belongs to the matchms filtering pipeline, not to load_from_mgf

## Inputs

- MGF-formatted MS/MS spectrum file (e.g. matchms tests/testdata/pesticides.mgf or all_your_spectrums.mgf)

## Outputs

- Iterator of matchms Spectrum objects (peaks + metadata), one per MGF spectrum entry

## How to apply

Call load_from_mgf on the MGF file (from matchms.importing), as in the paper's example workflow 'file = load_from_mgf("all_your_spectrums.mgf")'; the function returns an iterator of Spectrum objects, one per MGF entry. Iterate over the returned spectrums and collect, per spectrum, the number of peaks plus available metadata entries such as 'scans', 'ionmode', 'precursor_mz', and compound name if present. Optionally consult the pyteomics-based mzML/mzXML/MGF importer layer for format-specific parsing of the peak and metadata sections. Keep the raw peak arrays and the sparse metadata key-value pairs intact at this stage — the paper's separate metadata-cleaning filters (compound name, InChI, SMILES, ionmode, adduct, charge) run as a later stacked pipeline, not inside the import step.

## Related tools

- **matchms** (provides matchms.importing.load_from_mgf, the function used to parse MGF files into Spectrum objects) — https://github.com/matchms/matchms
- **pyteomics** (underlying layer that matchms's MGF importer is built on top of for file parsing)

## Examples

```
from matchms.importing import load_from_mgf; file = load_from_mgf("all_your_spectrums.mgf")
```

## Evaluation signals

- JSON record (parsed_mgf_spectra.json) exists and is valid JSON after parsing
- Spectrum count in the parsed output equals the number of entries in the input MGF file
- Each parsed spectrum's metadata keys include at least the fields present in the source MGF (scans, ionmode where annotated)
- Every parsed spectrum has a non-negative peak count consistent with the peaks listed in the corresponding MGF entry

## Limitations

- Large collected datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata — parsing alone does not fix this; metadata-cleaning filters must follow
- MGF is only one of several supported import formats; this skill does not cover mzML, mzXML, msp, JSON, or USI inputs
- Import yields raw Spectrum objects; peaks may need later normalization or select_by_mz / select_by_relative_intensity filtering before scoring

## Evidence

- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
- [abstract] MGF among import formats: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] metadata cleaning belongs to filters, not import: [evidence span withheld — anchor status: not_found]
- [abstract] pyteomics-based importers: "mzML, mzXML, MGF file importers are built on top of pyteomics"
- [abstract] inconsistent metadata in large datasets: "can further contain a significant fraction of inconsistent, wrongly labeled, or incorrect"
