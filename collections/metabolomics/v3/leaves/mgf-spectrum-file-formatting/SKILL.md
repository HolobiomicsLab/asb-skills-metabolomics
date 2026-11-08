---
name: mgf-spectrum-file-formatting
description: Use when pseudo MS/MS spectra have been generated from MS1-only LC-MS
  or MS imaging data and need to be written as MGF entries containing precursor feature
  m/z, peaks, and retention time/pixel metadata for annotation.
when_to_use_negative:
- Data already contains real MS/MS spectra — these should be annotated directly, not
  reformatted from MS1 clustering
- Only a feature table (m/z and intensity matrix) is available without per-feature
  chromatographic or spatial profiles, since clustering requires XIC or ion image
  correlation
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ms1-id
  role: Generates pseudo MS/MS spectra from correlated MS1 features and writes them
    to pseudo_msms.mgf in MGF format
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend for metabolic feature extraction and isotope peak flagging; isotope
    peaks are excluded before MGF writing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:4ed299d22cd5f3d80618d74be63d7aea739262e9526457ab3fa575897ba45567
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
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# MGF spectrum file formatting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formats clustered full-scan MS1 features (pseudo MS/MS spectra from LC-MS chromatographic or MS imaging spatial correlation) into MGF files with precursor feature m/z and retention time/pixel metadata, ready for downstream library matching.

## When to use

Use when pseudo MS/MS spectra have been generated from MS1-only LC-MS or MS imaging data and need to be written as MGF entries containing precursor feature m/z, peaks, and retention time/pixel metadata for annotation.

## When NOT to use

- Data already contains real MS/MS spectra — these should be annotated directly, not reformatted from MS1 clustering
- Only a feature table (m/z and intensity matrix) is available without per-feature chromatographic or spatial profiles, since clustering requires XIC or ion image correlation

## Inputs

- Clustered MS1 features from LC-MS (XIC-based correlation) or MS imaging (ion-image-based correlation)
- MassCube (ver 1.0.11) feature/isotope peak annotations
- MS1 peak heights as intensities

## Outputs

- pseudo_msms.mgf in MGF format with precursor feature m/z, retention time/pixel metadata, and correlated fragment peaks

## How to apply

After clustering MS1 features by Pearson correlation of XICs (Pearson >= 0.80, >= 4 shared consecutive MS1 scans) for LC-MS, or ion images (correlation >= 0.85, >= 50 shared non-zero pixels) for MS imaging, assemble each cluster's correlated ion m/z values with original MS1 peak heights as intensities into a pseudo MS/MS spectrum. Exclude isotope peaks determined by MassCube (ver 1.0.11) and include the precursor feature m/z plus retention time (or pixel) metadata in each entry. Write all spectra to a pseudo_msms.mgf file in MGF format, preserving precursor and peak fields so downstream tools (e.g., ms1_id annotate) can parse them. Validation requires the MGF to parse correctly and each entry to carry the precursor feature m/z and intensity-bearing peaks.

## Related tools

- **ms1-id** (Generates pseudo MS/MS spectra from correlated MS1 features and writes them to pseudo_msms.mgf in MGF format) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend for metabolic feature extraction and isotope peak flagging; isotope peaks are excluded before MGF writing)

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- pseudo_msms.mgf parses correctly as an MGF file
- Each entry contains a precursor feature m/z and correlated fragment peaks whose intensities derive from MS1 peak heights
- Isotope peaks flagged by MassCube are absent from the pseudo MS/MS peak lists
- Retention time or pixel metadata is present in each entry
- Applied correlation cutoffs (Pearson >= 0.80, >= 4 shared scans for LC-MS; correlation 0.85, >= 50 shared pixels for MSI) are consistent with the features in the MGF

## Limitations

- Pseudo MS/MS spectra reflect low-energy in-source fragments, which may mismatch reference MS/MS spectra collected at medium to high collision energies
- For MS imaging, fragments with m/z < 100 are usually removed because they are not typically observed in imaging data
- More than 40% of public LC-MS files are MS1-only, so coverage depends on the availability of correlated in-source fragments rather than true MS/MS

## Evidence

- [methods] Write all generated pseudo MS/MS spectra (with retention time / pixel metadata and precursor feature m/z) to pseudo_msms.mgf in MGF format.: [evidence span withheld — anchor status: not_found]
- [abstract] The method generates pseudo MS/MS spectra by clustering ions via correlation analysis of extracted ion chromatograms (XICs) in the retention time domain, or ion images in the spatial domain, treating both domains equivalently.: [evidence span withheld — anchor status: not_found]
- [readme] annotate pseudo MS/MS spectra: **mgf** files: [evidence span withheld — anchor status: external_doc]
