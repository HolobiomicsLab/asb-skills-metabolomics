---
name: metabolite-annotation-confidence-leveling
description: Use when input is MS1-only full-scan LC-MS data (mzML/mzXML, e.g. from
  Metabolomics Workbench) lacking MS/MS acquisition, and the goal is metabolite annotations
  at MSI level 2/3 confidence with per-mode counts.
when_to_use_negative:
- Input data already contains data-dependent MS/MS scans — direct MS/MS library search
  (e.g. MS2-id or GNPS search) is more appropriate than pseudo MS/MS reconstruction
- Input is MS imaging (imzML) data — use the ms1_id msi workflow with the stricter
  spatial-cutoff variant (minimum spectral usage 0.05, ≥50 shared pixels) instead
  of the LC-MS cutoffs
- Only molecular formula assignment is needed at MSI level 4/5 — tools such as METASPACE
  or formula-aware annotation are sufficient without spectral matching
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms1-id
  role: Python package that extracts metabolic features, generates pseudo MS/MS spectra
    from in-source fragments, and performs precursor-tolerant reverse spectral search
    against the indexed GNPS library to assign level 2/3 metabolite annotations
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend (v1.0.11) used by ms1-id for feature extraction, chromatographic correlation-based
    peak grouping, and isotope peak assignment (isotopes excluded from pseudo MS/MS
    generation)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:f3524a16c1e90a2ac0bd3a5403de5a681a6eaddb9485d629b44b5aa24ed1c690
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
yes_rate: 0.9
n_claims: 10
---

# metabolite-annotation-confidence-leveling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for producing level 2/3 structural metabolite annotations from MS1-only full-scan LC-MS data by generating pseudo MS/MS spectra and matching them against MS/MS libraries with ms1-id, then counting unique annotated metabolites per analytical mode.

## When to use

Use when input is MS1-only full-scan LC-MS data (mzML/mzXML, e.g. from Metabolomics Workbench) lacking MS/MS acquisition, and the goal is metabolite annotations at MSI level 2/3 confidence with per-mode counts.

## When NOT to use

- Input data already contains data-dependent MS/MS scans — direct MS/MS library search (e.g. MS2-id or GNPS search) is more appropriate than pseudo MS/MS reconstruction
- Input is MS imaging (imzML) data — use the ms1_id msi workflow with the stricter spatial-cutoff variant (minimum spectral usage 0.05, ≥50 shared pixels) instead of the LC-MS cutoffs
- Only molecular formula assignment is needed at MSI level 4/5 — tools such as METASPACE or formula-aware annotation are sufficient without spectral matching

## Inputs

- MS1-only full-scan LC-MS raw files in mzML or mzXML format (e.g. PR000639 IBD stool dataset, Q Exactive)
- Indexed MS/MS spectral library (e.g. GNPS library downloaded July 17, 2024, indexed with peak_scale_k=10)
- Metabolic feature table with retention time and precursor m/z per mode

## Outputs

- Level 2/3 metabolite annotation table per LC-MS mode with InChIKey, retention time, and precursor m/z
- Per-mode counts of unique annotated metabolites (e.g. ibd_ms1_annotation_counts.csv)
- Total count of unique level 2/3 metabolites across all modes

## How to apply

Download the full-scan LC-MS dataset (e.g. the 546-sample IBD stool project PR000639) and, for each LC-MS mode (HILIC positive, HILIC negative, C8 positive, C18 negative), run ms1-id with the MassCube backend (v1.0.11) to extract features and build pseudo MS/MS spectra: correlate features within a ±1.5 s retention-time window sharing at least 4 consecutive MS1 scans with Pearson correlation ≥ 0.80, excluding MassCube-assigned isotope peaks. Match each pseudo MS/MS spectrum against the indexed GNPS MS/MS library (July 17, 2024 version) using precursor-tolerant (open) reverse spectral search with square-root-transformed, mass-dependently peak-scaled reference spectra (scaling factor k = 10). Apply the LC-MS annotation cutoffs — minimum reverse cosine score 0.7, minimum 3 matched peaks, minimum spectral usage 0.20 — keeping the top 1 hit per unique precursor m/z. Link annotations back to metabolic features via retention time and precursor mass, then count unique InChIKey strings per mode and overall; for the IBD dataset this yields 3010, 293, 227, and 636 unique metabolites per mode, totaling 3802 at level 2/3 confidence.

## Related tools

- **ms1-id** (Python package that extracts metabolic features, generates pseudo MS/MS spectra from in-source fragments, and performs precursor-tolerant reverse spectral search against the indexed GNPS library to assign level 2/3 metabolite annotations) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend (v1.0.11) used by ms1-id for feature extraction, chromatographic correlation-based peak grouping, and isotope peak assignment (isotopes excluded from pseudo MS/MS generation))

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- For the IBD stool dataset (PR000639, 546 samples), per-mode unique metabolite counts should match 3010 (HILIC positive), 293 (HILIC negative), 227 (C8 positive), and 636 (C18 negative), with 3802 total unique level 2/3 metabolites
- Every retained annotation must satisfy the LC-MS cutoffs: reverse cosine ≥ 0.7, ≥ 3 matched peaks, spectral usage ≥ 0.20, and be the top 1 hit for its unique precursor m/z
- Pseudo MS/MS peak pairs must share ≥ 4 consecutive MS1 scans within a ±1.5 s RT window with Pearson correlation ≥ 0.80, and no MassCube-assigned isotope peaks should appear in pseudo spectra
- The indexed GNPS reference library must have been built with square-root intensity transform (peak_intensity_power 0.5) and peak_scale_k = 10; counts drop without peak scaling (demonstrated on 14 bile acid/drug standards)
- Counts table should be reproducible and saved as ibd_ms1_annotation_counts.csv with unique InChIKey strings per mode and overall

## Limitations

- Low-energy in-source fragments mismatch most public reference MS/MS spectra collected at medium to high collision energies; <1% of MoNA reference spectra were acquired at ≤5 eV, so peak scaling is required for valid matches
- Expected runtime is ~5–7 min per single LC-MS file; if longer than 10 min, increase --mass_detect_int_tol (default 2e5 for Orbitraps, 5e2 for QTOFs)
- Reverse spectral search score equation and mass-dependent peak scaling equation are not fully rendered in the article text, limiting exact reimplementation without the GitHub/Zenodo source

## Evidence

- [results] Applying ms1-id to the 546-sample IBD stool dataset in HILIC positive, HILIC negative, C8 positive, and C18 negative modes yielded 3010, 293, 227 and 636 unique metabolites respectively, totaling 3802 unique metabolites with level 2/3 confidence.: [evidence span withheld — anchor status: not_found]
- [readme] Expected runtime is ~5-7 min for a single LC-MS file. If it takes longer than 10 min, please increase the --mass_detect_int_tol parameter (default: 2e5 for Orbitraps, 5e2 for QTOFs).: [evidence span withheld — anchor status: not_found]
