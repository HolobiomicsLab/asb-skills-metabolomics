---
name: molecular-family-spectra-preprocessing
description: Use when raw MS/MS peak lists (e.g., from a GNPS2/FBMN task's specs_ms.mgf,
  keyed by scan number) must be cleaned and reduced before computing a global multiple-spectrum
  alignment of a molecular family.
when_to_use_negative:
- Input is already a preprocessed or aligned peak-set JSON from the MMSA sets-creation
  step (preprocessing has already been applied)
- Goal is display-side analysis (custom spectrum ordering, m/z range filtering, top-10
  intensity analysis, X-axis limits) rather than preparing spectra for alignment
- Analysis targets pairwise spectrum comparison or molecular network graph construction
  rather than multi-spectrum alignment within a family
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MMSA
  role: Web-based tool whose preprocessing layer performs the top-6 peak filtering
    (±25 m/z window, ±17 m/z precursor exclusion), square-root intensity transformation,
    and normalization before spectral alignment
  repo: github.com__Wang-Bioinformatics-Lab__NetworkFamily_MultipleAlignment_Website
- name: NetworkFamily_MultipleAlignment_Website
  role: Flask/Dash web application exposing the sets-creation interface where 'Process
    and Save JSON' generates the alignment JSON; core algorithms live in alignment.py
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: GNPS2
  role: Source of molecular networking tasks and spectra (specs_ms.mgf via USIs) whose
    spectra are fetched and preprocessed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:aafb49bfbe7af30560ddb67de382c82d55b9b6d177dcf2c8de8fb4bcd4328ec2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - NetworkFamily_MultipleAlignment_Website
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# molecular-family-spectra-preprocessing

## Summary

Preprocess raw MS/MS peak sets from a molecular network component before MMSA spectral alignment: filter each spectrum to its top 6 most intense peaks within a ±25 m/z range, exclude peaks within ±17 m/z of the precursor ion, apply square-root intensity transformation, and normalize peak intensities so downstream consensus-peak alignment is not dominated by outliers or precursor-region peaks.

## When to use

Use when raw MS/MS peak lists (e.g., from a GNPS2/FBMN task's specs_ms.mgf, keyed by scan number) must be cleaned and reduced before computing a global multiple-spectrum alignment of a molecular family.

## When NOT to use

- Input is already a preprocessed or aligned peak-set JSON from the MMSA sets-creation step (preprocessing has already been applied)
- Goal is display-side analysis (custom spectrum ordering, m/z range filtering, top-10 intensity analysis, X-axis limits) rather than preparing spectra for alignment
- Analysis targets pairwise spectrum comparison or molecular network graph construction rather than multi-spectrum alignment within a family

## Inputs

- Raw MS/MS peak sets per spectrum for a molecular family, keyed by scan number, from the sets-creation alignment JSON (e.g., scans referenced as mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>)
- Precursor ion m/z values for each spectrum (for the ±17 m/z precursor-exclusion rule)

## Outputs

- Structured JSON document of filtered peak sets keyed by scan number, with each retained spectrum's peak list containing at most 6 peaks that satisfy the ±17 m/z precursor-exclusion rule
- Square-root-transformed and normalized peak intensities ready for MMSA spectral alignment

## How to apply

Load the raw MS/MS peak sets for all spectra in the molecular family from the alignment JSON produced by the sets-creation step of the MMSA web application (input via GNPS Task ID, USIs, or FBMN, then 'Process and Save JSON'). For each spectrum, rank peaks by intensity and retain only the top 6 most intense peaks within a +/− 25 m/z range, dropping peaks close to the precursor ion (within ±17 m/z), since precursor-region signal carries no informative fragmentation for alignment. Then apply a square-root transformation to the retained peak intensities to reduce the effect of outliers, followed by normalization of peak intensities. Assemble the filtered, transformed, and normalized peak sets into a single structured JSON document keyed by scan number, which becomes the input for MMSA's global peak alignment across the family (as demonstrated on the rhamnolipid molecular family from P. aeruginosa LC-MS/MS data).

## Related tools

- **MMSA** (Web-based tool whose preprocessing layer performs the top-6 peak filtering (±25 m/z window, ±17 m/z precursor exclusion), square-root intensity transformation, and normalization before spectral alignment) — github.com__Wang-Bioinformatics-Lab__NetworkFamily_MultipleAlignment_Website
- **NetworkFamily_MultipleAlignment_Website** (Flask/Dash web application exposing the sets-creation interface where 'Process and Save JSON' generates the alignment JSON; core algorithms live in alignment.py) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **GNPS2** (Source of molecular networking tasks and spectra (specs_ms.mgf via USIs) whose spectra are fetched and preprocessed)

## Examples

```
python main.py  # launch NetworkFamily_MultipleAlignment_Website; input GNPS2 Task ID + component number (or USIs from specs_ms.mgf), then click 'Process and Save JSON' to emit the preprocessed alignment JSON
```

## Evaluation signals

- Output JSON exists, parses successfully, and is keyed by scan number
- Every retained spectrum's peak list contains at most 6 peaks
- No retained peak lies within ±17 m/z of the spectrum's precursor ion
- Retained peaks fall within the ±25 m/z intensity-ranking window used for top-6 selection
- Peak intensities in the output are square-root transformed and normalized (no raw intensity outliers dominating the distributions)

## Limitations

- Aggressive top-6 filtering can drop informative shifted peaks: MMSA did not capture the shifted peaks for the C12 HAA (m/z 239.162) or rhamnose bound to C12 (m/z 385.219) in the rhamnolipid family
- The greedy multiple alignment downstream of this preprocessing does not fully capture smaller peak alignments (e.g., y2 peaks in the albicidin family)

## Evidence
