---
name: isotopologue-peak-selection-targeted-ms
description: Use when quantifying a targeted metabolite and its isotopologues from
  LC-MS EICs where multiple peaks are detected per isotopologue and the correct target
  peak must be chosen via isotope-pattern similarity.
when_to_use_negative:
- Input is already a quantified feature/peak table with no chromatographic peak candidates
  — there is nothing to disambiguate via isotope-pattern similarity
- The analysis is untargeted feature detection rather than targeted metabolite/isotopologue
  quantification with a known molecular formula
- The compound's theoretical isotope pattern cannot be computed (no formula/resolution)
  or the metabolite is a single isotope peak with no isotopologue series to compare
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TarMet
  role: 'Implements the isotopologue extraction pipeline: airPLS baseline removal,
    MassSpecWavelet peak detection, Haar-wavelet bound estimation, and isotopic-similarity-based
    peak selection'
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Computes the theoretical isotope patterns of the target compounds at the MS
    instrument resolution for comparison against detected patterns
- name: airPLS
  role: Removes baselines of all EICs prior to peak detection
- name: MassSpecWavelet
  role: Detects peak positions in the baseline-corrected EICs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:c2a0bb7181c0e7a721446a463fff69bf06af9184078814554f393715f2ebc06e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# isotopologue-peak-selection-targeted-ms

## Summary

Selects the true target-metabolite peak among multiple chromatographic candidates by scoring each detected peak against the metabolite's theoretical isotope pattern (computed with enviPat at the instrument resolution) using TarMet's isotopic-similarity metric (Eq. 1), keeping the peak with maximum similarity.

## When to use

Use when quantifying a targeted metabolite and its isotopologues from LC-MS EICs where multiple peaks are detected per isotopologue and the correct target peak must be chosen via isotope-pattern similarity.

## When NOT to use

- Input is already a quantified feature/peak table with no chromatographic peak candidates — there is nothing to disambiguate via isotope-pattern similarity
- The analysis is untargeted feature detection rather than targeted metabolite/isotopologue quantification with a known molecular formula
- The compound's theoretical isotope pattern cannot be computed (no formula/resolution) or the metabolite is a single isotope peak with no isotopologue series to compare

## Inputs

- Extracted-ion chromatograms (EICs) for the target metabolite and its isotopologues from LC-MS runs
- Target metabolite definition (molecular formula/compound list used to compute theoretical patterns)
- MS instrument resolution (needed by enviPat to compute theoretical isotope patterns)

## Outputs

- Peak-position/bounds/area table per isotopologue annotated with per-peak isotopic-similarity values
- Selected target-metabolite peak (the candidate with maximum isotopic similarity)

## How to apply

Load the extracted-ion chromatograms (EICs) for the target metabolite and compute theoretical isotope patterns with enviPat at the MS instrument resolution. Remove each EIC baseline with airPLS, detect peak positions with the MassSpecWavelet package, and set peak bounds using a Haar-wavelet-detected valley between the target position and the neighboring peak with the largest intensity for improved SNR. Calculate peak areas for every detected peak candidate. For each candidate, compute the isotopic similarity of Eq. 1 between the theoretical isotope pattern and the detected isotope pattern across the isotopologues. Retain the candidate with the maximum isotopic similarity as the target-metabolite peak and emit the peak-position/bounds/area table annotated with per-peak similarity values. The rationale is that the true isotopologue series co-elutes with relative intensities matching the enviPat-predicted pattern, so pattern similarity disambiguates co-occurring interferences that intensity-based picking alone would misassign.

## Related tools

- **TarMet** (Implements the isotopologue extraction pipeline: airPLS baseline removal, MassSpecWavelet peak detection, Haar-wavelet bound estimation, and isotopic-similarity-based peak selection) — https://github.com/hcji/TarMet
- **enviPat** (Computes the theoretical isotope patterns of the target compounds at the MS instrument resolution for comparison against detected patterns)
- **airPLS** (Removes baselines of all EICs prior to peak detection)
- **MassSpecWavelet** (Detects peak positions in the baseline-corrected EICs)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- The retained target peak shows the maximum isotopic similarity (Eq. 1) among all candidate peaks for the isotopologue
- Peak bounds fall on a valley identified by the Haar wavelet between the target position and the largest-intensity neighboring peak
- Number of detected isotopologues and quantification quality should match the reported behavior in which TarMet detects more isotopologues and quantifies them better than state-of-the-art methods (e.g. on the MM14, human blood metabolites, and cellular metabolites datasets)
- Theoretical isotope patterns were generated with enviPet at the instrument resolution actually used, so detected relative intensities are comparable to predictions
- The per-peak similarity annotations in the output table are populated for every candidate, not only the selected peak

## Limitations

- Isotopic-similarity selection depends on enviPat-predicted patterns at the stated instrument resolution; incorrect resolution or formula leads to misassignment
- Peak-bound estimation relies on a Haar-wavelet valley between the target position and the single largest-intensity neighboring peak, which can fail for closely spaced or weak interference peaks
- As of the article/README, TarMet does not yet support data-independent acquisition or MS/MS library verification (listed as planned future features)

## Evidence
