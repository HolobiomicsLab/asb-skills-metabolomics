---
name: spectral-peak-filtering-and-normalization
description: Use when raw or unfiltered MS2 peak lists (.mgf/.mzML from GNPS jobs
  such as MSV000083437) must be filtered and normalized to GNPS defaults before recomputing
  or verifying cosine-similarity edges.
when_to_use_negative:
- Input spectra are already GNPS-filtered consensus spectra from the job output —
  re-applying the ±17 Da precursor and top-6-in-50-Da filters would alter the peak
  sets and shift cosine scores away from the published edges
- 'The goal is quantitative analysis: classic molecular networking is MS2-only and
  qualitative, so filtered spectra cannot support calibrated quantitation'
- The task is library identification rather than network-edge reproduction — then
  the 0.7 cosine / 6 matched peaks cutoffs apply to library search, not to peak filtering
  itself
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Defines and executes the peak-filtering parameters (precursor ion window ±17
    Da, top 6 peaks in ±50 Da windows) and the normalization used in molecular networking
    jobs
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw files to open formats (.mzXML, .mzML, .mgf)
    so peak lists can be filtered and normalized upstream of scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:cd49d96acce216082955b2d8c3bcf1239f262fe5462d1b7a1e1a85b896b37897
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MSConvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spectral-peak-filtering-and-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies GNPS-style MS2 peak filtering (precursor-window deletion and top-N peak selection) and base-peak normalization to consensus spectra (.mgf) so that downstream pairwise modified-cosine scoring reproduces the spectra actually compared by a GNPS molecular networking job.

## When to use

Use when raw or unfiltered MS2 peak lists (.mgf/.mzML from GNPS jobs such as MSV000083437) must be filtered and normalized to GNPS defaults before recomputing or verifying cosine-similarity edges.

## When NOT to use

- Input spectra are already GNPS-filtered consensus spectra from the job output — re-applying the ±17 Da precursor and top-6-in-50-Da filters would alter the peak sets and shift cosine scores away from the published edges
- The goal is quantitative analysis: classic molecular networking is MS2-only and qualitative, so filtered spectra cannot support calibrated quantitation
- The task is library identification rather than network-edge reproduction — then the 0.7 cosine / 6 matched peaks cutoffs apply to library search, not to peak filtering itself

## Inputs

- Consensus MS2 spectra in .mgf format with peak lists and precursor masses (e.g., downloaded from a GNPS networking job for MSV000083437)
- GNPS edge/network results table with node pair identifiers and cosine_score column
- GNPS networking job parameter settings (filter window, top-N, precursor mass tolerance)

## Outputs

- Filtered and base-peak-normalized peak lists per spectrum (top 6 peaks per 50 Da window, precursor ±17 Da region removed)
- Normalized peak vectors ready for modified-cosine pairwise scoring
- cosine_score_comparison.tsv with columns node pair, computed modified cosine score, GNPS cosine_score, and their difference

## How to apply

Parse the consensus .mgf file to obtain peak lists and precursor masses per spectrum, then apply the GNPS filter chain: (1) delete all peaks in a ±17 Da window around the precursor ion mass, (2) within each ±50 Da window retain only the top six most intense peaks (optional: mean + k × s.d. and minimum-peak-intensity cutoffs, which GNPS leaves off by default), and (3) normalize peak intensities to the base peak intensity so matched-peak intensities can be weighted (e.g., square-root intensity weighting during cosine scoring). These filters exist because structurally related molecules share fragment ion patterns, and removing chemically implausible precursor-region peaks prevents spurious matches. The filtered, normalized spectra are the direct inputs to modified-cosine pairwise alignment, where an empirical cutoff of cosine 0.7 with 6 MS2 ions matching is GNPS's default for library-style significance. Judge correctness by confirming the filter parameters match the networking job settings and that recomputed scores agree with the GNPS cosine_score column on the same node pairs.

## Related tools

- **GNPS** (Defines and executes the peak-filtering parameters (precursor ion window ±17 Da, top 6 peaks in ±50 Da windows) and the normalization used in molecular networking jobs) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw files to open formats (.mzXML, .mzML, .mgf) so peak lists can be filtered and normalized upstream of scoring)

## Examples

```
python filter_and_normalize.py --mgf MSV000083437_consensus.mgf --precursor-window 17 --top-n 6 --window 50 --normalize basepeak --out filtered_normalized.mgf
```

## Evaluation signals

- After filtering, no spectrum contains more than the top six most intense peaks within any ±50 Da window
- No retained peak falls within the ±17 Da window around the precursor ion mass
- All normalized peak intensities are scaled to base peak intensity (maximum normalized intensity = 1)
- Computed modified cosine scores on shared node pairs match the GNPS cosine_score column within a small tolerance when joined on node pair identifiers
- Edges below the GNPS default significance threshold (cosine 0.7 with 6 matched MS2 ions) are flagged consistently between computed and reference scores

## Limitations

- Filter Below Std Dev (mean + k × s.d.) and Minimum Peak Intensity filters are recommended off / default 0, so replicating a job requires checking its actual parameter values rather than assuming defaults
- Filtered, normalized spectra only support qualitative MS2 comparisons; quantitation requires follow-up targeted LC-MS
- The ±17 Da precursor-window and top-6-in-50-Da settings are job parameters — a networking job run with non-default settings will not be reproduced by default-value filtering

## Evidence

- [intro] An empirical cutoff for cosine scoring of 0.7 with 6 MS2 ions matching is the default setting in GNPS: [evidence span withheld — anchor status: not_found]
