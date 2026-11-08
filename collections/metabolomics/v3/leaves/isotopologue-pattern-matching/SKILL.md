---
name: isotopologue-pattern-matching
description: 'Use when performing targeted LC-MS stable-isotope-tracer or isotopologue
  analysis: you have defined target metabolites, extracted M+H isotopologue EICs per
  sample, and need to score observed isotope patterns against theoretical patterns
  at the MS resolution.'
when_to_use_negative:
- The goal is untargeted feature detection rather than targeted quantification of
  predefined metabolites and their isotopologues (TarMet is a targeted method).
- Data are already processed into a feature/intensity table without raw EICs, so baseline
  removal, peak picking with MassSpecWavelet, and RT alignment with PAFFT cannot be
  applied.
- MS/MS library verification or data-independent acquisition is needed — the TarMet
  README lists these only as planned future features.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: TarMet
  role: 'Reactive shiny GUI application that runs the whole isotopologue analysis
    workflow: target definition, EIC baseline removal (airPLS), peak detection (MassSpecWavelet),
    RT alignment (PAFFT), theoretical pattern calculation (enviPat), M+H isotopologue
    extraction with relative-abundance filtering (0.001), and isotopic similarity
    scoring'
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Computes the theoretical isotope patterns of the target compounds based on
    the resolution of the MS, which the observed isotopologue patterns are scored
    against
- name: airPLS
  role: Removes baselines of all the EICs prior to peak detection so isotopologue
    peak positions and abundances are accurately extracted
- name: MassSpecWavelet
  role: Detects peak positions in the baseline-corrected EICs, providing the candidate
    peaks that are filtered and matched to theoretical patterns
- name: PAFFT
  role: Aligns retention time shifts across samples so the same metabolite's isotopologues
    are matched consistently between samples
- name: AssayR
  role: Comparison tool used to benchmark isotopologue detection and isotopic similarity
    performance on the human blood metabolites dataset
  repo: https://gitlab.com/jimiwills/assay.R/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:27c60f9eac2cc06f40920db63828db4a0b1ed7f6341eaa5f86ae3f5215e3daff
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - AssayR
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# isotopologue-pattern-matching

## Summary

Match extracted-ion-chromatogram M+H isotopologue peaks to theoretical isotope patterns of target metabolites at the instrument resolution, using the isotopic similarity score to confirm and quantify targeted metabolites and their isotopologues in LC-MS data.

## When to use

Use when performing targeted LC-MS stable-isotope-tracer or isotopologue analysis: you have defined target metabolites, extracted M+H isotopologue EICs per sample, and need to score observed isotope patterns against theoretical patterns at the MS resolution.

## When NOT to use

- The goal is untargeted feature detection rather than targeted quantification of predefined metabolites and their isotopologues (TarMet is a targeted method).
- Data are already processed into a feature/intensity table without raw EICs, so baseline removal, peak picking with MassSpecWavelet, and RT alignment with PAFFT cannot be applied.
- MS/MS library verification or data-independent acquisition is needed — the TarMet README lists these only as planned future features.

## Inputs

- Raw LC-MS metabolomics data files (e.g. MM14, human blood metabolites, or cellular metabolites datasets) readable by mzR
- Target metabolite list (compound definitions/chemical formulas for theoretical isotope patterns)
- MS instrument resolution (needed by enviPat to generate theoretical isotope patterns)

## Outputs

- Per-metabolite isotopic similarity scores (observed vs theoretical isotope patterns, e.g. Table S2 equivalents)
- Detected isotopologue peak list with M+H isotopologue assignments per metabolite per sample
- Quantified isotopologue intensities/abundances for each target metabolite

## How to apply

Load the LC-MS dataset into TarMet and define the target metabolite list (e.g. 82 human blood metabolites). Preprocess each extracted-ion chromatogram: remove baselines with the airPLS method, detect peak positions with MassSpecWavelet, and align retention-time shifts across samples using PAFFT (Wong et al. 2005). Compute theoretical isotope patterns for each target compound at the instrument resolution via the enviPat package, then extract the M+H isotopologues and filter candidate peaks with a relative abundance threshold of 0.001. Finally, calculate the isotopic similarity between the observed and theoretical isotope patterns for each metabolite; this score both confirms metabolite identity and scales quantification of each isotopologue. The reactive shiny GUI lets you inspect per-metabolite scores and peak assignments interactively before exporting the isotopologue quantification table.

## Related tools

- **TarMet** (Reactive shiny GUI application that runs the whole isotopologue analysis workflow: target definition, EIC baseline removal (airPLS), peak detection (MassSpecWavelet), RT alignment (PAFFT), theoretical pattern calculation (enviPat), M+H isotopologue extraction with relative-abundance filtering (0.001), and isotopic similarity scoring) — https://github.com/hcji/TarMet
- **enviPat** (Computes the theoretical isotope patterns of the target compounds based on the resolution of the MS, which the observed isotopologue patterns are scored against)
- **airPLS** (Removes baselines of all the EICs prior to peak detection so isotopologue peak positions and abundances are accurately extracted)
- **MassSpecWavelet** (Detects peak positions in the baseline-corrected EICs, providing the candidate peaks that are filtered and matched to theoretical patterns)
- **PAFFT** (Aligns retention time shifts across samples so the same metabolite's isotopologues are matched consistently between samples)
- **AssayR** (Comparison tool used to benchmark isotopologue detection and isotopic similarity performance on the human blood metabolites dataset) — https://gitlab.com/jimiwills/assay.R/

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- Reproduce the article's benchmark: on the human blood metabolites dataset TarMet should detect all 76 of 82 metabolites with their isotopologues (AssayR detects fewer).
- Check the isotopic-similarity distribution: 56 metabolites should score > 0.90 and 69 should score > 0.80 (AssayR achieves only 36 and 61 respectively).
- Verify the candidate-peak filter: peaks below the relative abundance threshold of 0.001 must be excluded before isotopic similarity computation.
- Confirm theoretical patterns were generated with enviPat at the actual MS instrument resolution — pattern mismatch at wrong resolution will lower similarity scores.
- Inspect the GUI-assigned M+H isotopologue peak groups for consistent retention times across samples after PAFFT alignment.

## Limitations

- Performance is benchmarked on LC-MS targeted metabolomics datasets (MM14, human blood metabolites, cellular metabolites); behavior on other data types is not established.
- Data-independent acquisition and MS/MS library verification are not supported yet (planned future features).
- Detection quality depends on the relative abundance threshold (0.001) and correct MS resolution input for enviPat — mis-specified resolution degrades similarity scores.

## Evidence
