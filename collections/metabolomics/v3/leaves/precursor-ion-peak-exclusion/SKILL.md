---
name: precursor-ion-peak-exclusion
description: Use when preparing raw MS/MS peak lists for MMSA spectral alignment and
  uninformative peaks near the precursor ion (within ±17 m/z) must be excluded before
  top-6 intensity filtering and square-root transformation.
when_to_use_negative:
- Input is already a preprocessed or normalized feature table rather than raw MS/MS
  peak lists
- The analysis goal is pairwise spectral comparison outside a molecular family (e.g.,
  library matching), not MMSA multiple alignment preprocessing
- Peaks near the precursor ion are themselves the signal of interest (e.g., studying
  neutral-loss or in-source fragments), since the ±17 m/z rule discards them by design
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: Web-based tool whose sets-creation step applies the ±17 m/z precursor exclusion
    and top-6 peak filtering before multiple spectral alignment
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:a93932af8e7d8f98b1ff8d3ccd76cfeea179fb8708787e6447eb3c454208dd79
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - MMSA
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# precursor-ion-peak-exclusion

## Summary

A peak-filtering rule from MMSA preprocessing that removes MS/MS peaks lying within ±17 m/z of the precursor ion, applied alongside selecting the top 6 most intense peaks per ±25 m/z range before spectral alignment within a molecular family.

## When to use

Use when preparing raw MS/MS peak lists for MMSA spectral alignment and uninformative peaks near the precursor ion (within ±17 m/z) must be excluded before top-6 intensity filtering and square-root transformation.

## When NOT to use

- Input is already a preprocessed or normalized feature table rather than raw MS/MS peak lists
- The analysis goal is pairwise spectral comparison outside a molecular family (e.g., library matching), not MMSA multiple alignment preprocessing
- Peaks near the precursor ion are themselves the signal of interest (e.g., studying neutral-loss or in-source fragments), since the ±17 m/z rule discards them by design

## Inputs

- Raw MS/MS peak sets for spectra in a molecular family (e.g., specs_ms.mgf scans referenced via GNPS Task ID, USI, or FBMN task)
- Precursor ion m/z values for each spectrum
- Alignment JSON produced by the MMSA sets-creation step

## Outputs

- Filtered peak sets (≤6 peaks per spectrum after ±17 m/z precursor exclusion) assembled in a structured JSON keyed by scan number
- Preprocessed spectra ready for MMSA global spectral alignment

## How to apply

For each spectrum in the molecular family, rank peaks by intensity and select the top 6 most intense peaks within a ±25 m/z window, dropping any peak within ±17 m/z of the precursor ion since such near-precursor peaks carry little alignment value. The retained peaks then undergo square-root intensity transformation (to reduce outlier effects) and normalization of peak intensities before the global multiple-spectrum alignment is computed. Assemble the filtered peak sets into a structured JSON keyed by scan number via the MMSA web app's 'Process and Save JSON' step. Rationale: preprocessing 'includes filtering peaks, intensity transformation, and normalization of peak intensities' so that alignment focuses on fragment peaks that drive molecular networking relationships rather than residual precursor signal.

## Related tools

- **MMSA** (Web-based tool whose sets-creation step applies the ±17 m/z precursor exclusion and top-6 peak filtering before multiple spectral alignment) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Examples

```
gnproxy_url="mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>"; # enter USIs (one per line) in MMSA /setscreation/ then click "Process and Save JSON" to emit the filtered, precursor-excluded peak-set JSON
```

## Evaluation signals

- Each retained spectrum's peak list contains at most 6 peaks
- No retained peak lies within ±17 m/z of its spectrum's precursor ion m/z
- Output JSON exists, is parseable, and is keyed by scan number
- Downstream alignment proceeds only after square-root intensity transformation and normalization are applied to the filtered peaks

## Limitations

- The greedy multiple alignment downstream of this filtering does not fully capture smaller peak alignments (e.g., y2 peaks in the albicidin family)
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162) or rhamnose bound to C12 (m/z 385.219) in the rhamnolipid case study

## Evidence
