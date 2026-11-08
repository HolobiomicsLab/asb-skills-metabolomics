---
name: msms-peak-filtering
description: Use when raw MS/MS peak sets from spectra in a molecular network component
  must be preprocessed before MMSA multiple spectral alignment, e.g. to reduce noise
  and outlier intensity effects prior to consensus peak identification.
when_to_use_negative:
- The input is already an aligned consensus peak set or alignment JSON from a completed
  MMSA run — filtering at that stage would discard aligned signal.
- The fragments of interest lie close to the precursor ion (within ±17 m/z) or would
  not rank among the top 6 most intense peaks in a ±25 m/z window — they will be excluded
  by design.
- The goal is interactive analysis features such as m/z range limiting or top-10 intensity
  analysis of already-aligned data rather than preprocessing for alignment.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MMSA
  role: Web-based tool implementing the peak filtering, square-root intensity transformation,
    and normalization preprocessing prior to multiple mass spectral alignment
- name: NetworkFamily_MultipleAlignment_Website
  role: Flask/Dash web application whose sets-creation interface ('Process and Save
    JSON') produces the filtered peak-set JSON consumed by the alignment step
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:31dbcdb58839eb2fa5327f9ae1f38df881613d479bf90c218d3ef46a3cb98afd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - MMSA
  - NetworkFamily_MultipleAlignment_Website
  - NetworkX
  - Python Dash
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# msms-peak-filtering

## Summary

MMSA's preprocessing layer for MS/MS spectra within a molecular family: peak filtering (top 6 most intense peaks per ±25 m/z range, excluding peaks within ±17 m/z of the precursor), square-root intensity transformation, and normalization of peak intensities before spectral alignment.

## When to use

Use when raw MS/MS peak sets from spectra in a molecular network component must be preprocessed before MMSA multiple spectral alignment, e.g. to reduce noise and outlier intensity effects prior to consensus peak identification.

## When NOT to use

- The input is already an aligned consensus peak set or alignment JSON from a completed MMSA run — filtering at that stage would discard aligned signal.
- The fragments of interest lie close to the precursor ion (within ±17 m/z) or would not rank among the top 6 most intense peaks in a ±25 m/z window — they will be excluded by design.
- The goal is interactive analysis features such as m/z range limiting or top-10 intensity analysis of already-aligned data rather than preprocessing for alignment.

## Inputs

- Raw MS/MS peak sets (per-scan peak lists with m/z and intensity) for spectra in a molecular family, retrieved from GNPS2 task output clustering/specs_ms.mgf via Task ID or USI
- Precursor ion m/z values for each spectrum (needed for the ±17 m/z exclusion)
- Alignment JSON from the MMSA sets-creation step

## Outputs

- Filtered, square-root-transformed, and normalized peak sets per spectrum (at most 6 peaks per ±25 m/z window, none within ±17 m/z of the precursor)
- Structured JSON document of filtered peak sets keyed by scan number (written to ./temp/sets via 'Process and Save JSON')

## How to apply

Load the raw MS/MS peak sets for the spectra in the molecular family from the alignment JSON produced by the sets-creation step (e.g. from specs_ms.mgf scans fetched via GNPS2 task ID or USI). For each spectrum, rank peaks by intensity and select the top 6 most intense peaks within a ±25 m/z range, excluding peaks close to the precursor ion (within ±17 m/z) — this removes near-precursor and low-intensity noise peaks that do not contribute to family-level alignment. Then apply a square-root transformation to peak intensities to reduce the effect of outliers, followed by normalization of peak intensities so spectra are comparable in the modified-cosine-based MST alignment. Assemble the filtered peak sets into a single structured JSON keyed by scan number and persist it with 'Process and Save JSON' before computing alignments.

## Related tools

- **MMSA** (Web-based tool implementing the peak filtering, square-root intensity transformation, and normalization preprocessing prior to multiple mass spectral alignment)
- **NetworkFamily_MultipleAlignment_Website** (Flask/Dash web application whose sets-creation interface ('Process and Save JSON') produces the filtered peak-set JSON consumed by the alignment step) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Examples

```
python main.py  # launch the MMSA web app; in the /setscreation/ interface enter a GNPS2 Task ID + component number (or USIs) and click 'Process and Save JSON' to write filtered peak sets to ./temp/sets
```

## Evaluation signals

- Each retained spectrum's peak list contains at most 6 peaks satisfying the top-6 intensity rule within any ±25 m/z range.
- No retained peak lies within ±17 m/z of its spectrum's precursor ion m/z.
- The output JSON exists, is parseable, and is keyed by scan number with the filtered peak lists.
- Intensities in the output reflect the square-root transformation and normalization (compare raw vs. processed intensity ratios before and after processing).
- Downstream consensus sets on a known family reproduce published results, e.g. the rhamnolipid family consensus peaks at m/z 135.017, 211.130, 357.189, and 399.199.

## Limitations

- The greedy multiple alignment built on these filtered peak sets does not fully capture smaller peak alignments, such as the y2 peaks in the albicidin family.
- Because peaks within ±17 m/z of the precursor are excluded, informative near-precursor fragments are dropped before alignment.
- Filtering parameters are fixed by MMSA (top 6 peaks, ±25 m/z window, ±17 m/z precursor exclusion); only interactive analysis options (m/z range filtering, top-10 intensity analysis, spectrum selection) offer post-hoc flexibility.

## Evidence
