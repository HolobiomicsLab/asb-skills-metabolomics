---
name: spectral-intensity-rank-selection
description: Use when raw MS/MS peak lists (e.g., MGF specs_ms.mgf or alignment input
  JSON keyed by scan number) must be reduced to a small, comparable peak set per spectrum
  before computing consensus peaks or multiple spectral alignment.
when_to_use_negative:
- Input is already a processed/aligned peak set or a feature table (e.g., an FBMN
  feature quantification table) rather than raw per-spectrum peak lists
- The goal is interactive visualization-side filtering (m/z range limits, spectrum
  selection) rather than preprocessing for alignment — those are handled by the web
  tool's Advanced Filtering options
- Spectra are being analyzed with pairwise-only spectral matching rather than MMSA,
  where this top-6/±17 m/z preprocessing scheme is not defined
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3391
tools:
- name: MMSA
  role: Implements the preprocessing layer that applies the top-6 intensity-ranked
    peak filtering, precursor-exclusion, square-root intensity transformation, and
    normalization prior to spectral alignment
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkFamily_MultipleAlignment_Website
  role: Web application (Flask/Dash) whose sets-creation step loads raw MS/MS peak
    sets and generates the alignment JSON after 'Process and Save JSON'
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:5bc1adfeeeae40df77eb4936c3ed84a8369c4290e3835cc7a72bffd07953f4cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - MMSA
  - NetworkFamily_MultipleAlignment_Website
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-intensity-rank-selection

## Summary

A peak-filtering preprocessing step from the MMSA workflow that reduces raw MS/MS peak lists to the top 6 most intense peaks per m/z window before spectral alignment. It removes near-precursor noise and limits peak count so that downstream multiple-spectrum alignment of molecular families is not dominated by spurious or precursor-adjacent peaks.

## When to use

Use when raw MS/MS peak lists (e.g., MGF specs_ms.mgf or alignment input JSON keyed by scan number) must be reduced to a small, comparable peak set per spectrum before computing consensus peaks or multiple spectral alignment.

## When NOT to use

- Input is already a processed/aligned peak set or a feature table (e.g., an FBMN feature quantification table) rather than raw per-spectrum peak lists
- The goal is interactive visualization-side filtering (m/z range limits, spectrum selection) rather than preprocessing for alignment — those are handled by the web tool's Advanced Filtering options
- Spectra are being analyzed with pairwise-only spectral matching rather than MMSA, where this top-6/±17 m/z preprocessing scheme is not defined

## Inputs

- Raw MS/MS peak sets per spectrum (e.g., from specs_ms.mgf in an FBMN/GNPS2 task or retrieved by USI such as mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>)
- Precursor ion m/z values for each spectrum
- Alignment/sets-creation JSON keyed by scan number for the molecular family

## Outputs

- Filtered peak sets: at most 6 peaks per spectrum per ±25 m/z window, with peaks within ±17 m/z of the precursor removed
- Structured JSON document of filtered peaks keyed by scan number, ready for MMSA spectral alignment

## How to apply

Load the raw MS/MS peak sets for the spectra of a molecular family from the alignment JSON produced by the sets-creation step. For each spectrum, rank peaks by intensity and retain only the top 6 most intense peaks within a +/− 25 m/z range, excluding any peak within ±17 m/z of the precursor ion, since precursor-adjacent peaks are uninformative for inter-spectrum alignment. Follow this filtering with square-root intensity transformation (to reduce outlier effects) and peak intensity normalization before feeding the spectra into the MMSA global alignment. Assemble the filtered peak sets into a structured JSON document keyed by scan number for the alignment stage.

## Related tools

- **MMSA** (Implements the preprocessing layer that applies the top-6 intensity-ranked peak filtering, precursor-exclusion, square-root intensity transformation, and normalization prior to spectral alignment) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkFamily_MultipleAlignment_Website** (Web application (Flask/Dash) whose sets-creation step loads raw MS/MS peak sets and generates the alignment JSON after 'Process and Save JSON') — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Examples

```
git clone https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website && python main.py  # use 'Process and Save JSON' after entering a GNPS2 Task ID + component number
```

## Evaluation signals

- Each retained spectrum's peak list contains at most 6 peaks per ±25 m/z range
- No retained peak falls within ±17 m/z of the spectrum's precursor ion
- Output JSON exists, is parseable, and is keyed by scan number for every spectrum in the molecular family
- Downstream alignment runs only after the companion square-root intensity transformation and normalization steps are applied
- Filtering is applied uniformly across all spectra in the component, so the relative peak count structure is comparable across the family

## Limitations

- The greedy multiple alignment downstream of this filtering does not fully capture smaller peak alignments, such as y2 peaks in the albicidin family
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162) or rhamnose bound to C12 (m/z 385.219), so aggressive filtering plus greedy alignment can miss modified-core signals
- Excluding peaks within ±17 m/z of the precursor discards any genuine fragment information in that window

## Evidence
