---
name: mrm-targeted-metabolomics-peak-processing
description: Use when an MRM ion chromatogram contains coeluted peaks (e.g., 2–3 isomers
  sharing one transition, such as PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min)
  and per-compound abundances must be separated automatically.
when_to_use_negative:
- Peaks are well separated with no coelution — a simpler peak integration suffices
  and deconvolution adds no benefit
- A different tool already handles integration (e.g., TargetLynx or Skyline workflows)
  unless coelution handling is specifically needed; note TargetLynx overestimates
  coeluted compound abundance by attributing all coeluted peaks to one isomer
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Implements the automatic deconvolution of coeluted peaks within its peak detection
    and quantitation pipeline
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function used to find possible peak tip locations that seed
    the Gaussian models
- name: MATLAB
  role: Runtime environment in which the Gaussian fitting and deconvolution are executed
- name: Microsoft Excel
  role: Format of the quantitation output table receiving per-peak abundance results
- name: ProteoWizard
  role: msconvert converts vendor raw MRM data to the formats MRMQuant ingests
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:675a8f619242bf7f6ded2a5326fb38a520bd75c4363b2e5bdc1d1c03d9a50514
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
  - Microsoft Excel
  - ProteoWizard
  - findpeaks
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mrm-targeted-metabolomics-peak-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic deconvolution of coeluted peaks in MRM ion chromatograms by fitting Gaussian models at findpeaks-detected tip locations so the summed Gaussians minimize the difference with the extracted signal union, then integrating each fitted peak's height as the compound's abundance.

## When to use

Use when an MRM ion chromatogram contains coeluted peaks (e.g., 2–3 isomers sharing one transition, such as PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min) and per-compound abundances must be separated automatically.

## When NOT to use

- Peaks are well separated with no coelution — a simpler peak integration suffices and deconvolution adds no benefit
- A different tool already handles integration (e.g., TargetLynx or Skyline workflows) unless coelution handling is specifically needed; note TargetLynx overestimates coeluted compound abundance by attributing all coeluted peaks to one isomer

## Inputs

- Smoothed MRM ion chromatogram signal for the targeted compound region
- Candidate peak tip locations from MATLAB findpeaks
- Coeluted signal union across overlapping transitions
- MRM sample files and method file with targeted compound information (mzML or vendor-converted formats via ProteoWizard msconvert)

## Outputs

- Fitted Gaussian height (abundance) per deconvolved peak
- Per-peak abundance entries in the quantitation output table (Microsoft Excel xlsx file)

## How to apply

Load the smoothed ion chromatogram from the MRM sample file and the candidate peak tip locations identified by the MATLAB built-in function findpeaks. Construct the coeluted signal union across overlapping transitions for the targeted compound region. Initialize Gaussian models at the findpeaks tip locations and iteratively adjust each peak's height and sigma (width) so the sum of differences between the summed Gaussians and the extracted union signal is minimized. After background removal, integrate each fitted Gaussian signal — its height is extracted as that peak's abundance — and write per-peak results to the quantitation output (xlsx) table. This deconvolution is what allows MRMQuant to avoid the coelution failure modes seen in TargetLynx, which attributed all coeluted peaks to one compound and overestimated its abundance.

## Related tools

- **MRMQuant** (Implements the automatic deconvolution of coeluted peaks within its peak detection and quantitation pipeline) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function used to find possible peak tip locations that seed the Gaussian models)
- **MATLAB** (Runtime environment in which the Gaussian fitting and deconvolution are executed)
- **Microsoft Excel** (Format of the quantitation output table receiving per-peak abundance results)
- **ProteoWizard** (msconvert converts vendor raw MRM data to the formats MRMQuant ingests)

## Evaluation signals

- Each findpeaks tip location in a coeluted region is assigned its own Gaussian model — verify no coeluted peak is collapsed into a single abundance value
- The summed fitted Gaussians (after background removal) should closely match the extracted union signal (minimized sum of differences)
- Deconvolved peak abundances should appear as separate per-peak rows in the xlsx quantitation output
- On multi-isomer chromatograms (e.g., PC 34:3 isomers at RTs 4.72, 4.84, 4.92 min), individual isomer abundances should differ — one compound should not absorb the total signal
- Results should be inspectable in MRMQuant's visualization view for manual correction if required

## Limitations

- Assumes Gaussian peak shapes; coeluted peaks with non-Gaussian or unevenly sampled signals may fit poorly
- Depends on findpeaks correctly identifying all tip locations; missed tips will yield missed components
- Baseline estimation errors under coelution (as seen in TargetLynx connecting nearby valleys) can bias abundance — the union-signal fitting must adequately remove background

## Evidence

- [methods] Gaussian deconvolution objective: [evidence span withheld — anchor status: not_found]
- [abstract] findpeaks seeding: [evidence span withheld — anchor status: not_found]
