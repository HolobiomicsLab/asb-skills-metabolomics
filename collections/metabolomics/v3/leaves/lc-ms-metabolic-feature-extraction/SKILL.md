---
name: lc-ms-metabolic-feature-extraction
description: Use when raw LC-MS mzML/mzXML files (Orbitrap or Q-TOF, MS1-only or DDA)
  must be turned into metabolic features plus pseudo MS/MS spectra before spectral
  library annotation.
when_to_use_negative:
- Input is already a feature table or mgf pseudo MS/MS file — use the annotate mode
  instead of re-extracting features
- Data are imaging MS (imzML) — use the msi workflow rather than the LC-MS feature
  extraction
- Only true MS/MS spectra with high-energy fragments are needed — conventional MS/MS
  annotation without pseudo MS/MS generation suffices
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms1-id
  role: CLI driver that runs feature extraction (via MassCube) and generates pseudo
    MS/MS spectra from full-scan LC-MS data
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend used inside ms1-id (v1.0.11) to extract metabolic features and assign
    isotope peaks, which are excluded from pseudo MS/MS generation
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:32450b84773a06a9ea5a121783e63112731c7dfc65e6575037d0920e85a4f710
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# lc-ms-metabolic-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract metabolic features from full-scan LC-MS data (mzML/mzXML) with the MassCube backend inside ms1-id, generating the feature table and pseudo MS/MS spectra needed for downstream structural annotation.

## When to use

Use when raw LC-MS mzML/mzXML files (Orbitrap or Q-TOF, MS1-only or DDA) must be turned into metabolic features plus pseudo MS/MS spectra before spectral library annotation.

## When NOT to use

- Input is already a feature table or mgf pseudo MS/MS file — use the annotate mode instead of re-extracting features
- Data are imaging MS (imzML) — use the msi workflow rather than the LC-MS feature extraction
- Only true MS/MS spectra with high-energy fragments are needed — conventional MS/MS annotation without pseudo MS/MS generation suffices

## Inputs

- Raw LC-MS files in mzML or mzXML format (single or batched in a sample directory)
- Indexed MS/MS libraries (e.g. gnps.pkl and gnps_k10.pkl from the ms1_id releases)
- Mass backend version: MassCube ver 1.0.11

## Outputs

- aligned_feature_table.tsv with feature intensities and annotations
- Pseudo MS/MS (mgf) spectra per metabolic feature
- MS1 and MS/MS annotation result tables in the project directory

## How to apply

Run ms1-id's lcms command with the MassCube backend (ver 1.0.11) on a project directory containing raw mzML or mzXML files. MassCube extracts metabolic features and detects isotope peaks; features within a ±1.5 s retention-time window are correlated, requiring at least 4 consecutive shared MS1 scans and Pearson correlation ≥ 0.80 to link a fragment to its precursor feature. MassCube-assigned isotope peaks are excluded from pseudo MS/MS generation. Runtime is ~5–7 min per LC-MS file; if it exceeds 10 min, raise the --mass_detect_int_tol parameter (default 2e5 for Orbitraps, 5e2 for Q-TOFs). The aligned feature table (aligned_feature_table.tsv) and pseudo MS/MS spectra feed directly into the precursor-tolerant reverse spectral search against the indexed GNPS library.

## Related tools

- **ms1-id** (CLI driver that runs feature extraction (via MassCube) and generates pseudo MS/MS spectra from full-scan LC-MS data) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend used inside ms1-id (v1.0.11) to extract metabolic features and assign isotope peaks, which are excluded from pseudo MS/MS generation)

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- Feature correlation pairs share ≥ 4 consecutive MS1 scans with Pearson r ≥ 0.80 within a ±1.5 s RT window
- Pseudo MS/MS contain no MassCube-assigned isotope peaks
- Runtime per LC-MS file stays within ~5–10 min; exceeding 10 min indicates --mass_detect_int_tol must be raised (2e5 Orbitrap / 5e2 Q-TOF defaults)
- NIST fecal benchmark reproduces per-mode spectral-match counts (567 DDA 42 eV, 306 MS1 DDA, 484/511/604 at 0/10/20 eV) within ±10%
- IBD stool benchmark yields 3010/293/227/636 unique metabolites per mode (HILIC+, HILIC−, C8+, C18−) totaling 3802

## Limitations

- Feature extraction relies on correlation of chromatographic profiles, so no pseudo MS/MS is generated for features lacking correlated in-source fragments
- Q-TOF data may need a higher --mass_detect_int_tol because noise-level intensities differ from Orbitrap defaults (2e5 vs 5e2)
- Isotope peaks are only excluded if MassCube confidently assigns them; undetected isotopes may leak into pseudo MS/MS

## Evidence

- [discussion] For an MS1 peak to be paired with another, they must share at least 4 consecutive MS1 scans in their chromatographic profiles and the Pearson correlation coefficient between two XICs should be larger than 0.80 within a ±1.5 s retention time window: [evidence span withheld — anchor status: not_found]
- [readme] Expected runtime is ~5-7 min for a single LC-MS file. If it takes longer than 10 min, please increase the `--mass_detect_int_tol` parameter (default: 2e5 for Orbitraps, 5e2 for QTOFs).: [evidence span withheld — anchor status: external_doc]
- [readme] ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl: [evidence span withheld — anchor status: external_doc]
- [readme] Results can be accessed from `aligned_feature_table.tsv`.: [evidence span withheld — anchor status: external_doc]
