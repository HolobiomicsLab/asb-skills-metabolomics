---
name: chromatographic-feature-correlation-clustering
description: Use when input is MS1-only mzML/mzXML (or imzML) data lacking tandem
  spectra and the goal is to assemble in-source fragments correlated with a precursor
  into pseudo MS/MS spectra for structural annotation.
when_to_use_negative:
- Data already contains genuine MS/MS scans — direct library matching of real MS/MS
  spectra is preferable to pseudo-spectrum reconstruction.
- Input is an aligned feature table or intensity matrix without raw chromatographic/ion-image
  traces — correlation clustering needs per-feature XIC or ion-image profiles.
- Only the spectral-matching step is needed (already have MGF spectra) — use the annotation/matching
  skill instead of regenerating spectra.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Implements the correlation-based clustering of metabolic features (XIC or
    ion-image correlation) to produce pseudo MS/MS spectra in MGF format.
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend used for metabolic feature detection and isotope-peak flagging whose
    isotope annotations are excluded from pseudo MS/MS generation.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:38a40b6bb75cbadee9ab6cbb3fa86128298ff8da45fb0f8400e658a292c16195
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# chromatographic-feature-correlation-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates pseudo MS/MS spectra from MS1-only LC-MS (or MS imaging) data by clustering metabolic features via peak-peak correlation of extracted ion chromatograms or ion images, with isotope peaks excluded, producing MGF files ready for library matching.

## When to use

Use when input is MS1-only mzML/mzXML (or imzML) data lacking tandem spectra and the goal is to assemble in-source fragments correlated with a precursor into pseudo MS/MS spectra for structural annotation.

## When NOT to use

- Data already contains genuine MS/MS scans — direct library matching of real MS/MS spectra is preferable to pseudo-spectrum reconstruction.
- Input is an aligned feature table or intensity matrix without raw chromatographic/ion-image traces — correlation clustering needs per-feature XIC or ion-image profiles.
- Only the spectral-matching step is needed (already have MGF spectra) — use the annotation/matching skill instead of regenerating spectra.

## Inputs

- Full-scan (MS1-only) LC-MS data in mzML or mzXML format
- MS imaging data in imzML format (with accompanying ibd files)
- Metabolic feature list from MassCube feature detection (ver 1.0.11)

## Outputs

- pseudo_msms.mgf — MGF-format pseudo MS/MS spectra with precursor feature m/z and retention time / pixel metadata
- Per-precursor lists of correlated fragment ions with MS1-derived peak heights as intensities

## How to apply

Extract metabolic features from the mzML/mzXML input using the MassCube (ver 1.0.11) backend, then extract ion chromatograms (XICs) for each feature. For each pair of features within the same retention time window, compute peak-peak Pearson correlation on their chromatographic profiles, requiring the two ions to share at least 4 consecutive MS1 scans. For each target feature, collect all features with Pearson correlation coefficient >= 0.80 into a pseudo MS/MS spectrum, using original MS1 peak heights as fragment intensities, and exclude peaks flagged as isotope peaks by MassCube. For MS imaging input, instead extract ion images with ~10 ppm mass tolerance, require a minimum of 50 shared non-zero pixels between two ion images, and cluster ions using a spatial correlation cutoff of 0.85 after moving-average noise reduction and baseline setting within a moving 100 Da window (5× the mean intensity of the lowest 5% ions). Write all generated spectra, with precursor feature m/z and retention time or pixel metadata, to pseudo_msms.mgf in MGF format.

## Related tools

- **ms1-id** (Implements the correlation-based clustering of metabolic features (XIC or ion-image correlation) to produce pseudo MS/MS spectra in MGF format.) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend used for metabolic feature detection and isotope-peak flagging whose isotope annotations are excluded from pseudo MS/MS generation.)

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- The output MGF parses correctly and every entry contains a precursor feature m/z plus correlated fragment peaks whose intensities derive from original MS1 peak heights.
- No MassCube-flagged isotope peaks appear as fragments in the generated spectra.
- Applied cutoffs are consistent with the documented thresholds: Pearson correlation >= 0.80, >= 4 consecutive shared MS1 scans (or >= 50 shared non-zero pixels and spatial correlation 0.85 for imaging).
- For known-standard validation samples, pseudo MS/MS of a feature should recover fragments of the expected compound, enabling match scores >0.7 against reference libraries after peak scaling.
- Fragment ions larger than the precursor feature m/z should be absent when the spectra are used for modified-cosine molecular networking.

## Limitations

- Requires at least 4 consecutive shared MS1 scans between chromatographic profiles (or 50 shared non-zero pixels in imaging), so low-abundance or coeluting features with short overlap may not cluster.
- Low-energy in-source fragments in pseudo spectra often mismatch reference MS/MS collected at medium to high collision energies, which is why mass-dependent peak intensity scaling is applied downstream.
- Isotope-peak flagging depends on MassCube; features misassigned as isotopes would be wrongly excluded.
- For LC-MS, expected runtime is ~5-7 min per file; increasing --mass_detect_int_tol (default 2e5 Orbitrap, 5e2 QTOF) is recommended if longer.

## Evidence

- [results] The method generates pseudo MS/MS spectra by clustering ions via correlation analysis of extracted ion chromatograms (XICs) in the retention time domain, or ion images in the spatial domain: [evidence span withheld — anchor status: not_found]
- [discussion] Peaks that are determined as isotope peaks by MassCube are excluded from pseudo MS/MS generation: "Peaks that are determined as isotope peaks by MassCube are excluded from pseudo MS/MS generation"
- [discussion] they must share at least 4 consecutive MS1 scans in their chromatographic profiles: "they must share at least 4 consecutive MS1 scans in their chromatographic profiles"
- [discussion] A minimum of 50 shared pixels with non-zero intensities between two ion images is required: "A minimum of 50 shared pixels with non-zero intensities between two ion images is required"
- [readme] annotate LC-MS data: mzML or mzXML files; annotate imaging MS data: imzML files: [evidence span withheld — anchor status: not_found]
