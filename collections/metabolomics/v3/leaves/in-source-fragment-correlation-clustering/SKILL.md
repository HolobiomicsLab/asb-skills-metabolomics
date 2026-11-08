---
name: in-source-fragment-correlation-clustering
description: Use when input is full-scan (MS1-only) LC-MS data in mzML/mzXML — e.g.,
  files from GNPS/MassIVE lacking MS/MS — and the goal is to generate pseudo MS/MS
  spectra from in-source fragments for library-based structural annotation.
when_to_use_negative:
- Input already contains DDA MS/MS spectra for the features of interest — direct MS/MS
  spectral matching on real MS/MS is preferred; ms1-id's pseudo-spectrum route is
  for MS1-only data
- Data are MALDI imaging runs where fragments below m/z 100 are absent — use the gnps_minmz100
  indexed library and the MSI workflow instead of the LC-MS correlation pipeline
- Features lack sufficient chromatographic overlap (fewer than 4 consecutive shared
  MS1 scans or Pearson r < 0.80) — pseudo MS/MS cannot be reliably constructed for
  them
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms1-id
  role: Python package that performs correlation-based feature clustering, pseudo
    MS/MS generation, and reverse spectral matching for full-scan LC-MS annotation
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend (ver 1.0.11) used by ms1-id for metabolic feature extraction, peak
    picking, and isotope peak assignment whose isotope calls are excluded from pseudo
    MS/MS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:75b3b46255b3cb14f02e4ee67968e404575ddf818a3c7ecd69aa2aeb5bf4ee11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - MassCube
  - ms1-id
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# in-source-fragment-correlation-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters MS1 metabolic features by correlating extracted ion chromatograms so that in/post-source fragments co-eluting with a precursor can be merged into pseudo MS/MS (composite) spectra, enabling structural annotation of MS1-only LC-MS data via the ms1-id package.

## When to use

Use when input is full-scan (MS1-only) LC-MS data in mzML/mzXML — e.g., files from GNPS/MassIVE lacking MS/MS — and the goal is to generate pseudo MS/MS spectra from in-source fragments for library-based structural annotation.

## When NOT to use

- Input already contains DDA MS/MS spectra for the features of interest — direct MS/MS spectral matching on real MS/MS is preferred; ms1-id's pseudo-spectrum route is for MS1-only data
- Data are MALDI imaging runs where fragments below m/z 100 are absent — use the gnps_minmz100 indexed library and the MSI workflow instead of the LC-MS correlation pipeline
- Features lack sufficient chromatographic overlap (fewer than 4 consecutive shared MS1 scans or Pearson r < 0.80) — pseudo MS/MS cannot be reliably constructed for them

## Inputs

- MS1-only or full-scan LC-MS data files in mzML or mzXML format (e.g., NIST human fecal dataset MSV000095787)
- Pre-indexed MS/MS spectral library (e.g., GNPS library downloaded July 17, 2024, provided via Philipbear/ms1_id releases)

## Outputs

- Pseudo MS/MS (composite) spectra per metabolic feature, linking correlated in-source fragments to precursor m/z and retention time
- Feature table with spectral library annotations (aligned_feature_table.tsv from ms1-id lcms workflow), including per-feature matched compound hits

## How to apply

Run feature extraction and peak-picking with the MassCube backend (ver 1.0.11) inside ms1-id, then cluster features by correlating their extracted ion chromatograms: pair features within ±1.5 s retention time, require at least 4 consecutive shared MS1 scans, and accept a Pearson correlation ≥ 0.80 before merging a fragment into the precursor's pseudo MS/MS. Exclude peaks that MassCube determines to be isotope peaks from pseudo MS/MS generation, so composite spectra contain only genuine in-source fragments of the precursor. Treat chromatographic and spatial domains equivalently in the framework — for imaging data the same correlation logic is applied to ion images with a minimum of 50 shared non-zero pixels. The resulting pseudo MS/MS spectra are then matched against a peak-scaled (k = 10, square-root intensity) indexed GNPS library using precursor-tolerant reverse spectral search (minimum score 0.7, minimum 3 matched peaks, minimum spectral usage 0.20 for LC-MS). The rationale is that >40% of public LC-MS files are MS1-only, and their in/post-source fragments can act as handles for composite spectra that match reference MS/MS spectra.

## Related tools

- **ms1-id** (Python package that performs correlation-based feature clustering, pseudo MS/MS generation, and reverse spectral matching for full-scan LC-MS annotation) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend (ver 1.0.11) used by ms1-id for metabolic feature extraction, peak picking, and isotope peak assignment whose isotope calls are excluded from pseudo MS/MS)

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- Each pseudo MS/MS spectrum contains no peaks above its precursor m/z except when flagged; MassCube-assigned isotope peaks must be absent from generated pseudo spectra
- Correlated fragment–precursor pairs satisfy all three criteria: RT difference within ±1.5 s, ≥4 consecutive shared MS1 scans, and Pearson correlation ≥ 0.80
- Applying the full workflow to NIST reference human fecal data (MSV000095787) yields per-mode spectral match counts of 567 (DDA MS/MS 42 eV), 306 (MS1 DDA), 484 (MS1 0 eV), 511 (MS1 10 eV), and 604 (MS1 20 eV), each within ±10%
- Reverse spectral search results respect LC-MS cutoffs: minimum score 0.7, minimum 3 matched peaks, minimum spectral usage 0.20, with the top 1 hit kept per unique precursor m/z
- Annotation output should be consumable as the aligned_feature_table.tsv produced by ms1-id lcms runs

## Limitations

- Peak-peak correlation requires sufficient chromatographic sampling — features must share at least 4 consecutive MS1 scans, so fast gradients or sparse scans reduce pseudo MS/MS coverage
- Low-energy in-source fragments match reference MS/MS collected at medium-to-high collision energies poorly unless mass-dependent peak intensity scaling (k = 10, square-root transformation) is applied; <1% of MoNA reference spectra were acquired at ≤5 eV
- If an LC-MS annotation run takes longer than ~10 minutes per file, the --mass_detect_int_tol parameter (default 2e5 for Orbitraps, 5e2 for QTOFs) should be increased, indicating mass tolerance mismatch
- The correlation-based approach does not apply to data already acquired without any in-source fragmentation signal usable for clustering; enhanced isCID experimental designs need different handling (eISA-type methods)

## Evidence

- [readme] Library download instructions: [evidence span withheld — anchor status: external_doc]
