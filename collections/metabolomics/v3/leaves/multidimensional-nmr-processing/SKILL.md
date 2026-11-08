---
name: multidimensional-nmr-processing
description: Use when input is a time-domain multi-dimensional NMR FID dataset (possibly
  NUS-sampled, e.g. an NUScon Protein A HNCACB with 8.8% sampling schedule) and the
  goal is a processed, reconstructed 3D spectrum in NV format.
when_to_use_negative:
- Input is already a processed/frequency-domain spectrum needing only peak picking,
  assignment, chemical shift prediction, or structure calculation — use the corresponding
  NMRFx tools instead
- Data are uniformly (fully) sampled — GRINS NUS reconstruction is unnecessary; only
  apodization, zero-fill, FT, and phasing apply
- Data are 1D or other non-NMR formats not readable as NV files (magic value 874032077
  check will fail)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3452
tools:
- name: NMRFx Processor
  role: Auto-generates and executes the Python processing script; hosts the Processor
    accordion (apodization, zero-fill, FT) and GRINS NUS reconstruction step
  repo: github.com/nanalysis/nmrfx
- name: NMRFx
  role: Integrated application combining processing and visualization; updates the
    script interactively and executes on Process click
  repo: github.com/nanalysis/nmrfx
- name: NMRViewJ
  role: Used to open the reconstructed NV file and visually inspect planes against
    the reference spectrum
  repo: github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:035ea5c9b0e0b4bb046820955a8bda9d6687800e11178f5914fd6bcb902acb06
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  tools:
  - NMRFx
  - NMRFx Processor
  - NMRViewJ
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 22
---

# multidimensional-nmr-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Process multi-dimensional (e.g., 3D) biomolecular NMR datasets — including non-uniformly sampled ones — with NMRFx Processor, applying apodization, zero-fill, Fourier transform, and NUS reconstruction (GRINS) to produce NV-format spectra inspectable in NMRViewJ.

## When to use

Use when input is a time-domain multi-dimensional NMR FID dataset (possibly NUS-sampled, e.g. an NUScon Protein A HNCACB with 8.8% sampling schedule) and the goal is a processed, reconstructed 3D spectrum in NV format.

## When NOT to use

- Input is already a processed/frequency-domain spectrum needing only peak picking, assignment, chemical shift prediction, or structure calculation — use the corresponding NMRFx tools instead
- Data are uniformly (fully) sampled — GRINS NUS reconstruction is unnecessary; only apodization, zero-fill, FT, and phasing apply
- Data are 1D or other non-NMR formats not readable as NV files (magic value 874032077 check will fail)

## Inputs

- Multi-dimensional (3D) NMR time-domain FID dataset in NV format with NUS sampling schedule (e.g. NUScon Protein A HNCACB, synthetic peak set 1, 8.8% sampling)
- NUS sampling schedule defining measured hypercomplex points
- Auto-generated or hand-written Python processing script from NMRFx Processor

## Outputs

- Reconstructed 3D spectrum as NV file with 4-byte floating-point values in sub-matrix (bricked) layout
- Qualitative spectrum planes viewable in NMRViewJ for comparison with reference figures

## How to apply

Load the dataset in NV format and validate it by reading the first 4 bytes, which must yield the integer 874032077 (testing big-endian and little-endian modes to determine endianness). Use NMRFx Processor to auto-generate the Python processing routine, which appears as an interactive script linked to the Processor accordion and runs when the user clicks Process. Process each sampled FID plane through apodization, zero-fill, and Fourier transform along the directly detected dimension, then run the GRINS non-uniform sampling reconstruction to fill the ~91% of unmeasured hypercomplex points defined by the sampling schedule. Write the reconstructed 3D spectrum as an NV file with 4-byte floating-point values in sub-matrix (bricked) layout, letting Dataset.createDataFile() select BigMappedMatrixFile when total points × bytes exceed Integer.MAX_VALUE / 2, otherwise MappedSubMatrixFile; then open it in NMRViewJ and inspect planes.

## Related tools

- **NMRFx Processor** (Auto-generates and executes the Python processing script; hosts the Processor accordion (apodization, zero-fill, FT) and GRINS NUS reconstruction step) — github.com/nanalysis/nmrfx
- **NMRFx** (Integrated application combining processing and visualization; updates the script interactively and executes on Process click) — github.com/nanalysis/nmrfx
- **NMRViewJ** (Used to open the reconstructed NV file and visually inspect planes against the reference spectrum) — github.com/nanalysis/nmrfx

## Examples

```
# In NMRFx Processor GUI: load NUS HNCACB dataset, auto-generate Python script, then Process
# (script includes: proc.pdataset().apsf / zf / ft along direct dim, then GRINS reconstruction, then NV write)
```

## Evaluation signals

- Raw NV file passes the magic-value check: first 4 bytes decode to integer 874032077 in big- or little-endian mode
- Output NV file passes header checks: fileHeaderSize, blockElements, nDim, and per-dimension size/vsize fields are consistent with the 3D dataset
- GRINS reconstruction fills the ~91% of unmeasured hypercomplex points implied by the 8.8% sampling schedule
- Reconstructed planes qualitatively match the Fig. 1 reference NUS HNCACB spectrum of Protein A (expert visual review in NMRViewJ; no quantitative metric reported)

## Limitations

- Validation of the reconstruction is qualitative expert review only — no quantitative metric is reported in the text
- The full Python NUS processing script is only referenced as being in the SI, not reproduced in the article text
- No NUScon accession ID or download location is given for the specific HNCACB dataset, complicating direct reproduction
- NMRFx is under active development, so exact menus/scripting details may change between versions

## Evidence
