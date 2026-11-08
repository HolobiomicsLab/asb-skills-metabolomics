---
name: mrm-chromatogram-quantitation
description: Use when processing LC-MRM-MS targeted metabolomics data (e.g., bile-acid
  or phospholipid panels) where vendor raw files or mzML files plus a method file
  of targeted compounds must be converted into per-sample peak abundances, handling
  coeluted isomers and batch effects.
when_to_use_negative:
- When the MRM library file lacks the specified transitions — MRMPROBS-style failure
  shows this toolchain cannot proceed without matching transition records
- When the goal is general untargeted LC-MS peak picking rather than targeted MRM
  quantitation against a predefined compound list
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Primary tool performing peak detection, deconvolution, absolute quantitation
    and batch-effect correction on MRM chromatograms
  repo: https://github.com/kslynn128171/MRMQuant
- name: ProteoWizard
  role: Converts vendor raw mass-spectrometry data into mzML/mzXML readable by MRMQuant
- name: msconvert
  role: The ProteoWizard command-line utility invoked to produce the mzML input files
- name: MRMkit
  role: Comparison tool; its dataset (3793 samples, 33 batches, 295 compounds, 389
    BQCs) was used to validate MRMQuant batch correction
  repo: https://github.com/MRMkit/MRMkit
- name: findpeaks
  role: MATLAB built-in function used by MRMQuant to detect candidate peak tip locations
    in each ion chromatogram
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:9b14e5e9b8051d6625dbaf85a597e73eaac7f5d679b578fb6755162a23d99720
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
  - Microsoft Excel
  - ProteoWizard
  - findpeaks
  - msconvert
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# mrm-chromatogram-quantitation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Peak detection, deconvolution and absolute quantitation of MRM ion chromatograms from targeted LC-MS/MS runs, as implemented in MRMQuant. It converts vendor raw data to mzML, detects and deconvolves coeluted peaks, fits standard curves, and exports per-compound abundances in xlsx.

## When to use

Use when processing LC-MRM-MS targeted metabolomics data (e.g., bile-acid or phospholipid panels) where vendor raw files or mzML files plus a method file of targeted compounds must be converted into per-sample peak abundances, handling coeluted isomers and batch effects.

## When NOT to use

- When the MRM library file lacks the specified transitions — MRMPROBS-style failure shows this toolchain cannot proceed without matching transition records
- When the goal is general untargeted LC-MS peak picking rather than targeted MRM quantitation against a predefined compound list

## Inputs

- Vendor raw MRM data converted to mzML or mzXML via msconvert in ProteoWizard
- MRM method file listing targeted compounds with transitions
- Binary MATLAB MAT-file (generated intermediate, loadable into MRMQuant)
- Optional standard-curve sample files (e.g., 14 standard samples spanning 0.025–500 mol/mL)

## Outputs

- Microsoft Excel xlsx file with quantitated abundances per compound per sample
- Visualization of quantitated peaks for manual inspection and correction
- Batch-effect-corrected abundances when QC samples are included

## How to apply

Load MRM sample files (mzML or mzXML converted from vendor raw data with msconvert in ProteoWizard) plus a method file describing targeted compounds. MRMQuant uses the MATLAB built-in function findpeaks to locate possible peak tip locations, applies dynamic signal smoothing, and performs automatic deconvolution of coeluted peaks (e.g., PC 34:3 isomers at RTs 4.72, 4.84, 4.92 min). Quantitation supports absolute quantitation via standard curves and/or internal standards; saturated peaks are automatically excluded from the regression, and the user can skip non-fitting points by unchecking boxes in the regression plot. If QC samples (batch-pooled QCs, technical-replicate QCs) are present, batch-effect correction is applied automatically. Results are saved as a Microsoft Excel xlsx file; run MRMQuant v2.7 and optionally install get_PrecMZ_ProdMZ from the program/associated-programs folder to extract precursor and product m/z.

## Related tools

- **MRMQuant** (Primary tool performing peak detection, deconvolution, absolute quantitation and batch-effect correction on MRM chromatograms) — https://github.com/kslynn128171/MRMQuant
- **ProteoWizard** (Converts vendor raw mass-spectrometry data into mzML/mzXML readable by MRMQuant)
- **msconvert** (The ProteoWizard command-line utility invoked to produce the mzML input files)
- **MRMkit** (Comparison tool; its dataset (3793 samples, 33 batches, 295 compounds, 389 BQCs) was used to validate MRMQuant batch correction) — https://github.com/MRMkit/MRMkit
- **findpeaks** (MATLAB built-in function used by MRMQuant to detect candidate peak tip locations in each ion chromatogram)

## Evaluation signals

- Average STD of normalized abundances across eight technical-replicate QCs should be low (MRMQuant achieved 0.0832 without deconvolution, 0.0828 with automatic deconvolution, 0.0813 with reference correction over 163 retained phospholipids)
- Batch-effect correction should drive the BQC regression slope to ~0.0 (from 0.0004, R²=0.3 before correction to R²=1.44×10⁻⁴ after)
- Output must be a valid xlsx containing one abundance value per targeted compound per sample
- Number of unquantifiable compounds should be low — TargetLynx comparison showed high unquantifiable counts as a failure mode
- Manual inspection of displayed peaks should confirm coeluted isomers are attributed to the correct transitions, not merged into one compound

## Limitations

- MRMQuant processing time was around 5 h versus 4 h for MRMKit on the large 3793-sample dataset (wall-clock difference attributed to I/O, computation claimed comparable)
- Coeluted isomers of the same nominal compound must be correctly assigned; naive tools overestimate abundance by attributing all coeluted peaks to one compound
- MRMkit repository is archived; development moved to SLINGhub/MRMhub
- Uses MRMQuant v2.7; earlier versions may lack current features

## Evidence

- [readme] Repository maintenance status: [evidence span withheld — anchor status: external_doc]
