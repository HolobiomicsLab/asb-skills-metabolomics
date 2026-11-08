---
name: chromatographic-peak-tip-detection
description: Use when processing high-throughput LC-MS/MS MRM ion chromatograms where
  peaks may be coeluted, noisy, or unevenly sampled and per-transition peak tips,
  valleys, and coelution flags must be resolved before quantitation.
when_to_use_negative:
- Input is not an MRM ion chromatogram from targeted LC-MS/MS data (e.g. it is already
  a quantified peak table), since this skill operates on raw chromatographic signal
  during peak detection
- Only a single, isolated, symmetric peak is expected and no deconvolution or coelution
  handling is needed — simpler fixed-window peak picking may suffice
tools:
- name: MRMQuant
  role: Implements dynamic signal smoothing, findpeaks-based tip detection, HWHM/coelution
    bounding, and writes results to a binary MATLAB MAT-file for quantitation
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function called by MRMQuant to find possible peak tip locations
    in smoothed ion chromatograms
- name: MATLAB
  role: Runtime environment in which the MAT-file peak structure is generated and
    findpeaks is executed
- name: msconvert
  role: ProteoWizard tool used to convert vendor raw MRM data into the mzML format
    loaded for peak detection
- name: ProteoWizard
  role: Provides msconvert for converting raw vendor MRM data into the formats MRMQuant
    reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:dce141047f2a76e1f7d8614ac3680d4394660580913281ac4f132b33b098ec00
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
  - MATLAB
  - MRMQuant
  - ProteoWizard
  - findpeaks
  - msconvert
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatographic-peak-tip-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Locates chromatographic peak tips in MRM ion chromatograms using MATLAB's findpeaks after dynamic signal smoothing, tracing left/right bounds from each tip, reporting HWHM, and flagging/limiting coelution within 2.5× the half-width-at-half-maximum.

## When to use

Use when processing high-throughput LC-MS/MS MRM ion chromatograms where peaks may be coeluted, noisy, or unevenly sampled and per-transition peak tips, valleys, and coelution flags must be resolved before quantitation.

## When NOT to use

- Input is not an MRM ion chromatogram from targeted LC-MS/MS data (e.g. it is already a quantified peak table), since this skill operates on raw chromatographic signal during peak detection
- Only a single, isolated, symmetric peak is expected and no deconvolution or coelution handling is needed — simpler fixed-window peak picking may suffice

## Inputs

- MRM sample files (converted from vendor raw data to mzML or equivalent via msconvert in ProteoWizard)
- Method file containing targeted compound/transition information (precursor and product m/z, retention time)
- Per-ion chromatogram signal traces

## Outputs

- Per-transition peak tip locations
- Left- and right-hand peak bounds and half-width-at-half-maximum (HWHM)
- Left-/right-side coelution flags and split boundaries
- Binary MATLAB MAT-file peak structure consumed by later quantitation steps

## How to apply

For each ion chromatogram in the loaded MRM sample file, first apply dynamic signal smoothing, then call the MATLAB built-in function findpeaks to propose candidate peak tip locations. Trace each candidate tip upward until the signal declines to establish the peak-tip region, then use valley/shoulder criteria at the traced boundaries to decide whether the boundary is a genuine valley separating neighboring peaks or a shoulder of a coeluted peak, enabling automatic deconvolution of coeluted peaks. Compute the half-width-at-half-maximum (HWHM) for each candidate peak and flag left- or right-side coelution when a neighboring peak intrudes within a 2.5× HWHM bound on that side, restricting each traced bound to no larger than that 2.5× HWHM. Write the per-transition peak-tip locations, coelution flags, and split boundaries into the binary MATLAB MAT-file structure that MRMQuant uses for downstream absolute quantitation, batch-effect correction, and visualized inspection.

## Related tools

- **MRMQuant** (Implements dynamic signal smoothing, findpeaks-based tip detection, HWHM/coelution bounding, and writes results to a binary MATLAB MAT-file for quantitation) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function called by MRMQuant to find possible peak tip locations in smoothed ion chromatograms)
- **MATLAB** (Runtime environment in which the MAT-file peak structure is generated and findpeaks is executed)
- **msconvert** (ProteoWizard tool used to convert vendor raw MRM data into the mzML format loaded for peak detection)
- **ProteoWizard** (Provides msconvert for converting raw vendor MRM data into the formats MRMQuant reads)

## Evaluation signals

- For each transition, the detected peak tip aligns with the expected retention time of the targeted compound in the method file
- Left- and right-hand traced bounds terminate at local minima and never exceed 2.5× the computed HWHM on either side
- QC technical replicates yield consistent tip/valley assignments for the same transition, reflected in low replicate-to-replicate RT and STD variation (<0.1) as reported in the article
- Coeluted peaks (e.g. the three PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min) are attributed separately rather than merged into a single abundance
- The resulting MAT-file loads into MRMQuant and downstream quantitation, batch-effect correction, and visualized inspection proceed without peak-detection errors

## Limitations

- The traced bound on each side is capped at 2.5× the HWHM, so very broad or highly coeluted shoulders may still be clipped
- Boundaries must be internally consistent across QC replicates; if replicates disagree, the coelution splits or tips need manual correction via MRMQuant's visualized inspection
- Correct behavior on noisy and unevenly sampled peaks depends on the dynamic signal smoothing applied before findpeaks; skipping smoothing risks merging or dropping true peaks

## Evidence

- [abstract] findpeaks is used to find possible peak tip locations: [evidence span withheld — anchor status: not_found]
