---
name: mrm-ion-chromatogram-smoothing
description: Use when MRM sample files (mzML, MRM) must be preprocessed so that each
  ion chromatogram's peaks are detectable and traceable before peak detection, coelution
  deconvolution, and absolute quantitation in MRMQuant.
when_to_use_negative:
- Input data are already smoothed or the chromatogram is a flat baseline with no real
  peaks — dynamic smoothing would be unnecessary and could distort valleys between
  coeluted peaks.
- You are performing untargeted peak picking rather than targeted MRM quantitation
  with a method file specifying transitions; MRMQuant's smoothing is designed for
  targeted ion chromatograms, not de novo feature discovery.
- The workflow step needed is batch-effect correction across BQCs (the later pipeline
  stage); smoothing is a per-sample preprocessing step, not a cross-sample normalisation
  method.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Implements the dynamic signal smoothing step that precedes peak-tip detection
    on each ion chromatogram
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function that operates on the smoothed trace to find possible
    peak tip locations
- name: ProteoWizard
  role: Source of msconvert, used to convert vendor raw files into mzML/MRM input
    files MRMQuant can load
- name: msconvert
  role: Converter for vendor raw data into the mzML/MRM sample file formats MRMQuant
    reads
- name: MATLAB
  role: Runtime environment; the smoothed chromatograms are written to a binary MATLAB
    MAT-file
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:9715b5873f324efc26d8d2409e0a0752a125de3b2f961c6063365e2093dd83c5
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
  edam_operation: http://edamontology.org/operation_3215
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

# mrm-ion-chromatogram-smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Smoothing of MRM ion chromatograms with MRMQuant's dynamic signal smoothing (implemented in MATLAB) so that peak tips can be reliably located with the built-in findpeaks function before peak-boundary tracing and quantitation.

## When to use

Use when MRM sample files (mzML, MRM) must be preprocessed so that each ion chromatogram's peaks are detectable and traceable before peak detection, coelution deconvolution, and absolute quantitation in MRMQuant.

## When NOT to use

- Input data are already smoothed or the chromatogram is a flat baseline with no real peaks — dynamic smoothing would be unnecessary and could distort valleys between coeluted peaks.
- You are performing untargeted peak picking rather than targeted MRM quantitation with a method file specifying transitions; MRMQuant's smoothing is designed for targeted ion chromatograms, not de novo feature discovery.
- The workflow step needed is batch-effect correction across BQCs (the later pipeline stage); smoothing is a per-sample preprocessing step, not a cross-sample normalisation method.

## Inputs

- MRM sample files in mzML or MRM format (converted from vendor raw data via msconvert/ProteoWizard)
- Method file with targeted compound information (transition precursor/product m/z, RT)

## Outputs

- Smoothed per-ion-chromatogram signal stored in the binary MATLAB MAT-file structure used by MRMQuant for peak detection and quantitation
- Candidate peak tips, boundaries, and coelution flags derived from the smoothed traces

## How to apply

Load each MRM ion chromatogram extracted from the sample files (converted from vendor raw data with msconvert/ProteoWizard) into MRMQuant and apply its dynamic signal smoothing step, which is the first of the pipeline's five features and precedes peak detection. The smoothed trace is what the MATLAB built-in 'findpeaks' function is then run on to find possible peak tip locations, so smoothing quality directly determines whether tips, valleys, and coeluted-peak boundaries can be recovered; noisy peaks and peaks with unevenly sampled signals are exactly the failure cases the smoothing must fix. Smoothing is per-sample and automatic (minimal human intervention), with the result stored in the binary MATLAB MAT-file that downstream quantitation, batch-effect correction, and visualized inspection steps read. Only re-run smoothing if the initial quantitation output shows missed tips or merged coeluted peaks; the manual inspection and correction GUI can then be used to check specific ion chromatograms.

## Related tools

- **MRMQuant** (Implements the dynamic signal smoothing step that precedes peak-tip detection on each ion chromatogram) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function that operates on the smoothed trace to find possible peak tip locations)
- **ProteoWizard** (Source of msconvert, used to convert vendor raw files into mzML/MRM input files MRMQuant can load)
- **msconvert** (Converter for vendor raw data into the mzML/MRM sample file formats MRMQuant reads)
- **MATLAB** (Runtime environment; the smoothed chromatograms are written to a binary MATLAB MAT-file)

## Examples

```
mrq = MRMQuant('samples', glob('*.mzML'), 'method', 'targets.csv'); % dynamic signal smoothing runs automatically before findpeaks
```

## Evaluation signals

- After smoothing, findpeaks must report peak tip locations consistent with the known transition RTs in the method file; a missed tip or a spurious tip on noise indicates smoothing failed.
- Coeluted isomer pairs (e.g., the PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min) must remain separable after smoothing, not merged into a single peak envelope.
- Across QC technical replicates, the smoothed traces should yield low replicate-to-replicate RT and STD variation (<0.1 was the observed benchmark in the article) — instability signals over- or under-smoothing.
- Peaks with noisy or unevenly sampled signal must still produce usable peak boundaries after smoothing; failure here is the exact pathology the smoothing step is meant to prevent.
- The output MAT-file must be loadable by MRMQuant so that downstream absolute quantitation (standard curves and/or internal standards) and batch-effect correction proceed without error.

## Limitations

- Dynamic signal smoothing cannot recover a peak if the raw signal is absent (saturated peaks are automatically excluded from regression later, so their abundances are unreliable regardless of smoothing).
- Smoothing is a preprocessing step only; it does not itself resolve coelution — the deconvolution of coeluted peaks is a separate downstream feature of MRMQuant.
- The article reports that even with smoothing, tools like MRMkit and Skyline had problematic quantitation of noisy peaks, coeluted peaks, and peaks with unevenly sampled signals; smoothing reduces but does not eliminate these failure modes.
- Be sure to use the latest version of MRMQuant (currently v2.7); the README warns about version currency.

## Evidence

- [abstract] findpeaks used on smoothed trace: [evidence span withheld — anchor status: not_found]
