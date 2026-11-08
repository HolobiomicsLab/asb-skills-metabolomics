---
name: grins-nus-reconstruction
description: Use when a 3D NMR dataset is non-uniformly sampled (e.g. an NUScon schedule
  such as 8.8% sampling) and the unmeasured hypercomplex points must be reconstructed
  before spectral inspection in NMRViewJ.
when_to_use_negative:
- The dataset is uniformly sampled — no unmeasured hypercomplex points exist, so standard
  apodization/zero-fill/Fourier transform in NMRFx Processor suffices
- You need a quantitatively scored reconstruction quality metric; the paper reports
  only expert qualitative comparison against Fig. 1 with no quantitative metric
tools:
- name: NMRFx
  role: Integrated application whose Processor accordion hosts the interactive Python
    script containing the GRINS NUS reconstruction step
  repo: github.com/nanalysis/nmrfx
- name: NMRFx Processor
  role: Auto-generates the Python processing routine and executes apodization, zero-fill,
    Fourier transform and GRINS reconstruction when Process is clicked
- name: NMRViewJ
  role: Opens the reconstructed NV file to inspect planes against the Fig. 1 reference
    spectrum
- name: GRINS
  role: Non-uniform sampling reconstruction algorithm that fills the ~91% unmeasured
    hypercomplex points
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:d611b3ab8365f242cbf029951bd45f16ec2f83eceff81fdc9def3bc856a73618
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  tools:
  - GRINS
  - NMRFx
  - NMRFx Processor
  - NMRViewJ
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# grins-nus-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct non-uniformly sampled (NUS) multidimensional NMR spectra with the GRINS algorithm inside NMRFx Processor's Python-driven processing pipeline, reproducing the NUScon Protein A HNCACB spectrum from the NMRFx paper.

## When to use

Use when a 3D NMR dataset is non-uniformly sampled (e.g. an NUScon schedule such as 8.8% sampling) and the unmeasured hypercomplex points must be reconstructed before spectral inspection in NMRViewJ.

## When NOT to use

- The dataset is uniformly sampled — no unmeasured hypercomplex points exist, so standard apodization/zero-fill/Fourier transform in NMRFx Processor suffices
- You need a quantitatively scored reconstruction quality metric; the paper reports only expert qualitative comparison against Fig. 1 with no quantitative metric

## Inputs

- NUScon Protein A HNCACB NUS FID dataset in NV format (synthetic peak set 1, 8.8% sampling schedule)
- NMRFx Processor auto-generated Python processing script (referenced in the SI)

## Outputs

- Reconstructed 3D HNCACB spectrum written as an NV file with 4-byte floating-point values in bricked/sub-matrix layout
- Qualitative match of reconstructed planes against the Fig. 1 reference spectrum

## How to apply

Load the NUS FID in NV format and validate by reading the first 4 bytes, which must yield the integer 874032077 (testing big- and little-endian modes to determine endianness). Use NMRFx Processor to auto-generate the Python processing routine, then process each sampled FID plane through apodization, zero-fill, and Fourier transform along the directly detected dimension. Run the GRINS non-uniform sampling reconstruction step to fill the ~91% of unmeasured hypercomplex points defined by the 8.8% sampling schedule; the script updates interactively with the Processor accordion and executes when the user clicks Process. Write the reconstructed 3D spectrum as an NV file with 4-byte floating-point values in sub-matrix (bricked) layout via Dataset.createDataFile(), which picks BigMappedMatrixFile if total points converted to bytes exceed Integer.MAX_VALUE/2, otherwise MappedSubMatrixFile.

## Related tools

- **NMRFx** (Integrated application whose Processor accordion hosts the interactive Python script containing the GRINS NUS reconstruction step) — github.com/nanalysis/nmrfx
- **NMRFx Processor** (Auto-generates the Python processing routine and executes apodization, zero-fill, Fourier transform and GRINS reconstruction when Process is clicked)
- **NMRViewJ** (Opens the reconstructed NV file to inspect planes against the Fig. 1 reference spectrum)
- **GRINS** (Non-uniform sampling reconstruction algorithm that fills the ~91% unmeasured hypercomplex points)

## Evaluation signals

- First 4 bytes of the input NV file read as integer 874032077 (big- or little-endian)
- Output NV file passes magic-value and header checks: fileHeaderSize, blockElements, nDim, and per-dimension size/vsize fields
- Reconstructed planes qualitatively match the Fig. 1 reference spectrum upon expert review in NMRViewJ
- Correct storage backend selected: BigMappedMatrixFile only when total points converted to bytes exceed Integer.MAX_VALUE/2, otherwise MappedSubMatrixFile

## Limitations

- Full Python NUS processing script is only referenced as being in the SI, not provided in the article text
- No quantitative reconstruction-quality metric is reported; validation is expert qualitative comparison
- No NUScon accession ID or download location is given for the specific HNCACB dataset
- NMRFx software version is not stated

## Evidence

- [discussion] workflow validation detail: [evidence span withheld — anchor status: not_found]
