---
name: pseudo-msms-spectrum-generation
description: Use when input is MS1-only data — mzML/mzXML LC-MS runs or imzML imaging
  acquisitions — and structural (level 2/3) annotation requires pseudo MS/MS spectra
  built from correlated in-source fragments.
when_to_use_negative:
- Data already contains true MS/MS scans (DDA MS/MS spectra) — use direct MS/MS library
  matching instead of pseudo-spectrum reconstruction.
- Input is already a pseudo_msms.mgf file — proceed directly to annotation rather
  than regenerating spectra.
- The target features' co-eluting ions do not share ≥ 4 consecutive MS1 scans (LC-MS)
  or ≥ 50 non-zero shared pixels (imaging), so no correlated cluster can be formed.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Python package that performs the correlation-based clustering of features/ions
    to build pseudo MS/MS spectra from LC-MS and imaging MS full-scan data, and annotates
    the resulting MGF files
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Feature-detection backend used to extract metabolic features, generate the
    XIC profiles used for peak-peak correlation, and flag isotope peaks that are excluded
    from pseudo MS/MS generation
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:002ce1695fd403c8eda88c9a304d354b9332dd985b1b3e56b82055aa941e0f28
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
  - Numba
  - ms1-id
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# pseudo-msms-spectrum-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate pseudo MS/MS spectra from MS1-only LC-MS or MS imaging data by clustering in/post-source fragment ions that correlate with a target feature across the chromatographic (XIC) or spatial (ion-image) domain, then write them as MGF for downstream spectral-library annotation with ms1-id.

## When to use

Use when input is MS1-only data — mzML/mzXML LC-MS runs or imzML imaging acquisitions — and structural (level 2/3) annotation requires pseudo MS/MS spectra built from correlated in-source fragments.

## When NOT to use

- Data already contains true MS/MS scans (DDA MS/MS spectra) — use direct MS/MS library matching instead of pseudo-spectrum reconstruction.
- Input is already a pseudo_msms.mgf file — proceed directly to annotation rather than regenerating spectra.
- The target features' co-eluting ions do not share ≥ 4 consecutive MS1 scans (LC-MS) or ≥ 50 non-zero shared pixels (imaging), so no correlated cluster can be formed.

## Inputs

- MS1-only LC-MS raw files in mzML or mzXML format
- MS imaging data in imzML (with ibd) format
- Feature table from the MassCube feature-detection backend

## Outputs

- pseudo_msms.mgf — pseudo MS/MS spectra in MGF format with precursor feature m/z and retention-time or pixel metadata
- Peak intensities derived from original MS1 peak heights

## How to apply

For LC-MS, extract metabolic features with the MassCube (ver 1.0.11) backend, then for each feature pair within the same retention-time window (e.g. ±1.5 s) compute peak-peak Pearson correlation on extracted ion chromatograms, requiring at least 4 consecutive shared MS1 scans; collect features with Pearson correlation ≥ 0.80 into a pseudo MS/MS spectrum using original MS1 peak heights as intensities, excluding peaks MassCube flags as isotopes. For MS imaging, preprocess each scan with moving-average noise reduction (100 Da window, baseline = 5× mean of the lowest 5% ions in the window), centroid if needed, extract ion images at ~10 ppm mass tolerance, require a minimum of 50 shared non-zero pixels between two ion images, and cluster ions with a spatial correlation cutoff of 0.85 — the framework treats chromatographic and spatial domains equivalently. Write the resulting spectra with retention-time or pixel metadata and precursor feature m/z to pseudo_msms.mgf in MGF format. The rationale is that >40% of public LC-MS files and nearly all MS imaging files are MS1-only, so in/post-source fragments captured within the MS1 scan serve as the only fragmentation evidence available for reverse spectral matching.

## Related tools

- **ms1-id** (Python package that performs the correlation-based clustering of features/ions to build pseudo MS/MS spectra from LC-MS and imaging MS full-scan data, and annotates the resulting MGF files) — https://github.com/Philipbear/ms1_id
- **MassCube** (Feature-detection backend used to extract metabolic features, generate the XIC profiles used for peak-peak correlation, and flag isotope peaks that are excluded from pseudo MS/MS generation)

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- The output MGF parses correctly and each entry contains a precursor feature m/z plus correlated fragment peaks whose intensities derive from MS1 peak heights.
- Applied LC-MS cutoffs (Pearson correlation ≥ 0.80, ≥ 4 consecutive shared MS1 scans) and imaging cutoffs (≥ 50 shared non-zero pixels, spatial correlation 0.85) are recorded in the run parameters.
- Isotope peaks flagged by MassCube are absent from the generated pseudo MS/MS spectra.
- Downstream reverse spectral matching of the generated pseudo MS/MS reproduces reported match counts on NIST fecal data (e.g. 306 MS1-DDA and 604 MS1-20 eV features), within ±10% tolerance.
- For imaging data (e.g. MTBLS313 mouse brain), generated pseudo MS/MS supports annotation of PC(16:0/18:1) in [M+H]+ and [M+Na]+ forms with reverse cosine scores ≈0.85 and 0.77.

## Limitations

- Isotope peaks are excluded from pseudo MS/MS generation, so fragment clusters containing isotopic series may be trimmed.
- The spatial-correlation path requires ≥ 50 shared non-zero pixels between two ion images, so features absent in more than half of the imaged area cannot cluster.
- Peak scaling with mass-dependent factor k = 10 and square-root intensity transformation is applied at the library side, not during pseudo-spectrum generation itself; low-energy in-source fragments may still mismatch high-collision-energy reference spectra if scaling is skipped.

## Evidence

- [readme] The README confirms ms1_id is the tool that annotates pseudo MS/MS MGF files and builds indexed MS/MS libraries.: [evidence span withheld — anchor status: external_doc]
