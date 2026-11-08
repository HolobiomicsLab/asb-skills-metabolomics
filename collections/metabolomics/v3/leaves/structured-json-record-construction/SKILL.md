---
name: structured-json-record-construction
description: Use when you have loaded spectrums from an MGF-formatted file via matchms.importing.load_from_mgf
  and need a structured, verifiable JSON record of each spectrum's peaks and metadata
  before downstream filtering or scoring.
when_to_use_negative:
- The data are already in mzML, mzXML, msp, JSON, or USI form and you want a different
  importer's parsing behavior — load_from_mgf is specific to MGF files.
- The goal is metadata cleaning or scoring rather than a raw import inventory — use
  matchms filter functions (e.g. default_filters, require_minimum_number_of_peaks)
  or the Pipeline class instead.
- You need spectra comparison outputs (cosine or modified cosine scores), not per-spectrum
  records.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: matchms
  role: Provides matchms.importing.load_from_mgf to parse the MGF file into an iterator
    of Spectrum objects whose peaks and metadata are collected into the JSON record;
    installable via conda from anaconda.org/nlesc/matchms or from GitHub source.
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Underlying importer library for matchms (mzML, mzXML, MGF file importers are
    built on top of pyteomics), so MGF parsing behavior inherits pyteomics conventions.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:3e79e415fc1681b9d5f47d58ea021a8f1040c8ece114b080670bdb51c8aa1c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structured-json-record-construction

## Summary

Iterate over Spectrum objects returned by matchms.importing.load_from_mgf and serialize per-spectrum peak counts and metadata (scans, ionmode, precursor_mz, compound name) into a structured JSON record such as parsed_mgf_spectra.json, giving a machine-readable, validated snapshot of an MGF import.

## When to use

Use when you have loaded spectrums from an MGF-formatted file via matchms.importing.load_from_mgf and need a structured, verifiable JSON record of each spectrum's peaks and metadata before downstream filtering or scoring.

## When NOT to use

- The data are already in mzML, mzXML, msp, JSON, or USI form and you want a different importer's parsing behavior — load_from_mgf is specific to MGF files.
- The goal is metadata cleaning or scoring rather than a raw import inventory — use matchms filter functions (e.g. default_filters, require_minimum_number_of_peaks) or the Pipeline class instead.
- You need spectra comparison outputs (cosine or modified cosine scores), not per-spectrum records.

## Inputs

- MGF-formatted spectrum file (e.g. matchms package test data such as tests/testdata/pesticides.mgf)
- Spectrum objects yielded by matchms.importing.load_from_mgf

## Outputs

- parsed_mgf_spectra.json — valid JSON with one entry per imported spectrum containing peak count and metadata fields (scans, ionmode, precursor_mz, compound name where annotated)

## How to apply

Call matchms.importing.load_from_mgf on the MGF file (e.g. a file from the matchms package's tests/testdata directory, such as pesticides.mgf) to obtain an iterator of Spectrum objects. Iterate over the spectrums and, for each, collect the number of peaks plus the metadata entries present (e.g. 'scans', 'ionmode', 'precursor_mz', compound name if present). Write the collected counts and metadata fields into a structured JSON record saved as parsed_mgf_spectra.json, one entry per imported spectrum. Rationale: matchms is designed so raw data can be imported from common formats including MGF, and recording the pre-filter state of the metadata makes later cleaning/harmonizing filter steps auditable — especially since large MS/MS datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata.

## Related tools

- **matchms** (Provides matchms.importing.load_from_mgf to parse the MGF file into an iterator of Spectrum objects whose peaks and metadata are collected into the JSON record; installable via conda from anaconda.org/nlesc/matchms or from GitHub source.) — https://github.com/matchms/matchms
- **pyteomics** (Underlying importer library for matchms (mzML, mzXML, MGF file importers are built on top of pyteomics), so MGF parsing behavior inherits pyteomics conventions.)

## Examples

```
from matchms.importing import load_from_mgf; spectrums = [s for s in load_from_mgf("all_your_spectrums.mgf")]; records = [{"n_peaks": len(s.peaks), "metadata": dict(s.metadata)} for s in spectrums]; import json; json.dump(records, open("parsed_mgf_spectra.json", "w"))
```

## Evaluation signals

- parsed_mgf_spectra.json exists and is valid JSON.
- The JSON contains one entry per imported spectrum, and the spectrum count equals the number of entries in the input MGF file.
- Each entry's metadata keys include at least the fields present in the source MGF (e.g. scans, ionmode where annotated).
- Per-spectrum peak counts are non-negative integers consistent with the corresponding Spectrum object's peaks.
- Re-running load_from_mgf on the same MGF file reproduces an identical JSON record (deterministic import).

## Limitations

- MGF is only one of matchms's supported import formats; msp, mzML, mzXML, JSON, and USI require different importers.
- Large collected MS/MS datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, so the recorded fields reflect the raw (pre-cleaning) state and may be wrong.
- Some metadata fields (e.g. ionmode, compound name) may simply be absent from the source MGF, so the record must tolerate missing keys rather than requiring them.

## Evidence

- [other] The documented matchms example workflow imports spectrums from an MGF formatted file using the load_from_mgf function from matchms.importing: [evidence span withheld — anchor status: not_found]
- [abstract] Raw data can be imported from the commonly used formats msp, mzML, mzXML, MGF: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] large collected datasets contain data acquired using different instruments and measurement conditions, and can further contain a significant fraction of inconsistent, wrongly labeled, or incorrect: "large collected datasets contain data acquired using different instruments and measurement conditions, and can further contain a significant fraction of inconsistent, wrongly labeled, or incorrect"
- [readme] below is an example of using default filter steps for cleaning spectra, followed by calculating the Cosine score between mass Spectra in the tests/testdata/pesticides.mgf: [evidence span withheld — anchor status: external_doc]
