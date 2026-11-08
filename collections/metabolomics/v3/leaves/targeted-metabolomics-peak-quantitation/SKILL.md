---
name: targeted-metabolomics-peak-quantitation
description: Use when MRM chromatogram files (mzML or NetCDF from vendor raw data)
  plus a method file of targeted compounds need automated peak detection, coeluted-peak
  deconvolution, and absolute quantitation with optional QC-based batch-effect correction.
when_to_use_negative:
- Input is already a quantified abundance/feature table — re-running peak detection
  would be redundant.
- No method file with targeted compound transitions and RTs exists — targeted quantitation
  cannot proceed reliably.
- The goal is purely to flatten batch drift in BQC regression (slope/R2) after abundances
  are computed — use QC-RLSC batch-effect correction rather than re-quantitation.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0593
tools:
- name: MRMQuant
  role: 'Primary tool: automated MRM peak detection, deconvolution, absolute quantitation,
    and batch-effect correction'
  repo: https://github.com/kslynn128171/MRMQuant
- name: ProteoWizard
  role: Converts vendor raw data to mzML/NetCDF input formats via msconvert
- name: findpeaks
  role: MATLAB built-in function used inside MRMQuant to find possible peak tip locations
- name: MRMkit
  role: Benchmark comparator for quantitation quality; its data set (3793 samples,
    33 batches, 295 compounds, 389 BQCs) is used to demonstrate before/after correction
  repo: https://github.com/MRMkit/MRMkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:bcf4cf724cc10b09b9cda1baa12e660e461ff42cf7481d68389e72211320793e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MRMQuant
  - MRMkit
  - ProteoWizard
  - findpeaks
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# targeted-metabolomics-peak-quantitation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated peak detection, deconvolution, and quantitation of LC-MRM chromatograms (mzML/NetCDF) against a method file of targeted compounds, using MRMQuant's MATLAB findpeaks-based peak detection, dynamic signal smoothing, coeluted-peak deconvolution, and absolute quantitation via standard curves and/or internal standards.

## When to use

Use when MRM chromatogram files (mzML or NetCDF from vendor raw data) plus a method file of targeted compounds need automated peak detection, coeluted-peak deconvolution, and absolute quantitation with optional QC-based batch-effect correction.

## When NOT to use

- Input is already a quantified abundance/feature table — re-running peak detection would be redundant.
- No method file with targeted compound transitions and RTs exists — targeted quantitation cannot proceed reliably.
- The goal is purely to flatten batch drift in BQC regression (slope/R2) after abundances are computed — use QC-RLSC batch-effect correction rather than re-quantitation.

## Inputs

- MRM sample files in mzML or NetCDF (converted from vendor raw data with msconvert in ProteoWizard)
- Method file with targeted compound information (transitions, precursor/product m/z, RTs)
- Standard samples at known concentrations (e.g., 0.025-500 mol/mL) for standard-curve absolute quantitation
- Optional QC samples (e.g., batch-pooled QCs/BQCs) for batch-effect correction

## Outputs

- Quantitation results in a Microsoft Excel xlsx file
- Binary MATLAB MAT-file loadable into MRMQuant
- Per-compound abundances with regression plots for visualized inspection and correction

## How to apply

Convert vendor raw files to mzML or NetCDF with msconvert (ProteoWizard), then load the MRM sample files and a method file with the targeted compound information into MRMQuant; optionally use the get_PrecMZ_ProdMZ associated program to acquire precursor and product m/z values. Run peak detection, where the MATLAB built-in function findpeaks finds possible peak tip locations, followed by dynamic signal smoothing and automatic deconvolution of coeluted peaks so that all coeluted isomer peaks (e.g., three PC 34:3 isomers at RT 4.72, 4.84, and 4.92 min) are not attributed to a single compound. Quantitate absolute abundance via standard curves (e.g., standard samples spanning 0.025-500 mol/mL) and/or internal standards, then run batch-effect correction if QC samples are included, with saturated peaks automatically excluded from the regression. Inspect quantitated peaks visually and manually correct or skip points (e.g., by unchecking ion chromatogram boxes) if they do not fit the curve.

## Related tools

- **MRMQuant** (Primary tool: automated MRM peak detection, deconvolution, absolute quantitation, and batch-effect correction) — https://github.com/kslynn128171/MRMQuant
- **ProteoWizard** (Converts vendor raw data to mzML/NetCDF input formats via msconvert)
- **findpeaks** (MATLAB built-in function used inside MRMQuant to find possible peak tip locations)
- **MRMkit** (Benchmark comparator for quantitation quality; its data set (3793 samples, 33 batches, 295 compounds, 389 BQCs) is used to demonstrate before/after correction) — https://github.com/MRMkit/MRMkit

## Evaluation signals

- Every quantitated peak maps to a targeted compound in the method file, with precursor/product m/z values verifiable via get_PrecMZ_ProdMZ.
- Coeluted isomers are resolved, not merged: e.g., the PC 34:3 ion chromatogram yields three isomer peaks at RT 4.72, 4.84, and 4.92 min instead of attributing all coeluted peaks to one compound.
- On technical-replicate QC data (8 replicates, 184 phospholipids), replicate abundances show low average STD and RT differences (well-behaved tools achieved <0.1).
- Outputs are a valid xlsx quantitation file and a MATLAB MAT-file that loads into MRMQuant.
- If QC samples are included, BQC regression against injection order should flatten after correction (e.g., MRMkit data set: slope 0.0004/R2 0.3 before vs slope ~0.0/R2 1.44e-4 after).

## Limitations

- Coeluted, noisy, and unevenly sampled peaks are problematic for several existing tools — MRMkit and Skyline showed problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals, while OpenMS could not consistently quantify the same peak when multiple peaks existed near the specified RT.
- Baseline estimation errors can occur in other tools under coelution: TargetLynx attributed all coeluted peaks to PC 34:3 and overestimated its abundance, and its valley-connecting baseline underestimated abundance with high coelution in the right valley.
- Manual inspection and correction of quantitated peaks may still be required after automated quantitation.

## Evidence

- [abstract] findpeaks used for peak tip detection: [evidence span withheld — anchor status: not_found]
