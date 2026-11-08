---
name: metabolomics-feature-table-mgf-preparation
description: Use when MZmine-preprocessed full-scan and MS/MS data (e.g. from RegFilter-scheduled
  precursor lists) must be exported as a feature table (.csv) and spectra (.mgf) for
  FBMN-based annotation of ions missed by DDA.
when_to_use_negative:
- Input is already a MZmine-exported feature table and MGF — proceed directly to FBMN/annotation
  instead of re-preprocessing
- 'Data were not acquired by LC-MS/MS with MS2 spectra (e.g. MS1-only runs), since
  ''Export/Submit to GNPS-FBMN: Filter rows: Only with MS2'' would discard all features'
- The analysis targets DDA-fragmented precursors only, rather than recovering list-only
  ions missed by DDA fragmentation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MSConvert
  role: Converts raw MS files to .mzXML format before MZmine preprocessing
- name: MZmine 2.53
  role: Preprocesses full scan and MS/MS data (mass detection, ADAP, deconvolution,
    alignment) and exports the feature table (.csv) and GNPS-FBMN .mgf
- name: RegFilter
  role: Produces the scheduled precursor list whose features must be covered by the
    exported feature table and list-only spectra
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:febfa0b3fe444f88d6a97e8f41336cc95c688837e94b907f63153002f8e3a05b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - GNPS
  - GNPS2
  - MSConvert
  - MZmine
  - MZmine 2.53
  - RegFilter
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolomics-feature-table-mgf-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares LC-MS/MS metabolomics outputs — an MZmine feature quantification table (.csv) and a list-only fragmentation spectra file (.mgf) — so they can be annotated downstream via GNPS2 FBMN molecular networking, ChemWalker propagation, and SIRIUS in silico annotation.

## When to use

Use when MZmine-preprocessed full-scan and MS/MS data (e.g. from RegFilter-scheduled precursor lists) must be exported as a feature table (.csv) and spectra (.mgf) for FBMN-based annotation of ions missed by DDA.

## When NOT to use

- Input is already a MZmine-exported feature table and MGF — proceed directly to FBMN/annotation instead of re-preprocessing
- Data were not acquired by LC-MS/MS with MS2 spectra (e.g. MS1-only runs), since 'Export/Submit to GNPS-FBMN: Filter rows: Only with MS2' would discard all features
- The analysis targets DDA-fragmented precursors only, rather than recovering list-only ions missed by DDA fragmentation

## Inputs

- Raw LC-MS/MS files in .mzXML format (converted with MSConvert) from full scan and MS/MS modes
- MZmine feature quantification table (.csv) with row ID, row m/z, row retention time
- Fragmentation spectra exported as .mgf via MZmine's Export/Submit to GNPS-FBMN module

## Outputs

- Feature quantification table (.csv) containing filtered features with row ID, m/z, and retention time
- MGF spectra file of fragmentation spectra paired to feature table rows, ready for GNPS2 FBMN submission

## How to apply

Convert raw MS files to .mzXML with MSConvert, then preprocess full scan and MS/MS modes separately in MZmine 2.53. Run MZmine steps with the paper's parameters: mass detection MS1 (Centroid, noise 1E3) and MS2 (Centroid, noise 1E2), ADAP (min group size 4, group intensity threshold 1E3, min highest intensity 3E3, m/z tolerance 0.001 m/z or 15 ppm), chromatogram deconvolution (Baseline cut-off, min peak height 3E3, peak duration 0–15), isotopic peak grouping (0.001 m/z or 15 ppm, RT tolerance 0.2, max charge 3), join aligner (0.001 m/z or 15 ppm, weights m/z 75 / RT 25), and peak finder (intensity tolerance 10, 0.001 m/z or 15 ppm, RT tolerance 0.2). Export the feature quantification table to .csv with row ID, m/z, and retention time, and use 'Export/Submit to GNPS-FBMN' with 'Filter rows: Only with MS2' to generate the .mgf of fragmentation spectra. These paired files are the required inputs for the GNPS2 FBMN workflow, ChemWalker propagation ([M+H]+, 15 ppm), and SIRIUS (v5.8.5) annotation.

## Related tools

- **MSConvert** (Converts raw MS files to .mzXML format before MZmine preprocessing)
- **MZmine 2.53** (Preprocesses full scan and MS/MS data (mass detection, ADAP, deconvolution, alignment) and exports the feature table (.csv) and GNPS-FBMN .mgf)
- **RegFilter** (Produces the scheduled precursor list whose features must be covered by the exported feature table and list-only spectra) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
conda activate regfilter && pip install git+https://github.com/computational-chemical-biology/regression_filter.git && jupyter notebook notebooks/building_spl_and_assessing_XIC_for_all_ions.ipynb
```

## Evaluation signals

- Feature table .csv includes the exported common elements: row ID, row m/z, and row retention time
- MGF export used 'Filter rows: Only with MS2' so every exported feature has associated MS2 spectra
- Consistent tolerances across MZmine steps: m/z tolerance 0.001 m/z or 15 ppm and RT tolerance 0.2 for isotopic grouping, aligner, and peak finder
- Downstream FBMN on GNPS2 yields per-strain list-only ion annotation counts matching reported values (BRA006: 43, BRA010: 46, BRA177: 47)
- RegFilter report confirms the number of total features and filtered features (135 for BRA006, 219 for BRA010, 150 for BRA177) are representable in the exported table

## Limitations

- Not all detected ions are fragmented in DDA data acquisition; the MGF may contain spectra only for list-only ions, requiring spectral pairing to recover annotations
- MZmine 2.53 is an older version; parameters (e.g. ADAP, baseline cut-off deconvolution) are version-specific and may need adjustment for other versions
- Ions with low m/z and low relative abundance are captured by the SPL but must be handled by downstream annotation layers (ChemWalker, SIRIUS), not by the export step itself

## Evidence
