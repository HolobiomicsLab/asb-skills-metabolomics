---
name: targeted-metabolite-extraction
description: Use when LC-MS/centWave chromatograms of a defined list of target metabolites
  must be quantified at the isotopologue level, with isotope-pattern similarity scores
  per adduct (M+H, M+Na, M+K) across samples.
when_to_use_negative:
- The goal is untargeted feature discovery rather than quantification of a predefined
  metabolite list — TarMet requires targets to be defined up front.
- Input is already a processed feature/peak table without raw extracted ion chromatograms,
  since airPLS baseline removal and MassSpecWavelet peak detection operate on EICs.
- Data are data-independent acquisition (DIA) or require MS/MS library verification
  — both are listed as planned future features, not supported.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TarMet
  role: Reactive shiny GUI application that orchestrates the entire targeted metabolite
    and isotopologue quantification workflow
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Provides TarMet's reactive graphical user interface
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Calculates theoretical isotope patterns of target compounds based on MS resolution
- name: airPLS
  role: Removes baselines of all extracted ion chromatograms before peak detection
- name: MassSpecWavelet
  role: Detects peak positions of the isotopologues in the baseline-corrected EICs
- name: PAFFT
  role: Aligns retention-time shifts across samples to match targeted isotopologues
    (Wong et al. 2005)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:5bfae0013c6c12f2c5b85428398a8bd39cc578220ded75edcdd1400f4c9c39fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# targeted-metabolite-extraction

## Summary

Targeted extraction and quantification of isotopologues (M+H, M+Na, M+K adducts) of known metabolites from LC-MS chromatograms using the TarMet shiny application, combining enviPat theoretical isotope patterns, airPLS baseline correction, MassSpecWavelet peak detection, and PAFFT retention-time alignment.

## When to use

Use when LC-MS/centWave chromatograms of a defined list of target metabolites must be quantified at the isotopologue level, with isotope-pattern similarity scores per adduct (M+H, M+Na, M+K) across samples.

## When NOT to use

- The goal is untargeted feature discovery rather than quantification of a predefined metabolite list — TarMet requires targets to be defined up front.
- Input is already a processed feature/peak table without raw extracted ion chromatograms, since airPLS baseline removal and MassSpecWavelet peak detection operate on EICs.
- Data are data-independent acquisition (DIA) or require MS/MS library verification — both are listed as planned future features, not supported.

## Inputs

- centWave-processed LC-MS chromatograms / extracted ion chromatograms (e.g. the MM14 dataset from http://msbi.ipb-halle.de/msbi/centwave)
- List of target metabolites (molecular formulas/structures, e.g. 14 standard compounds)
- MS resolution setting used to derive theoretical isotope patterns

## Outputs

- Peak table of detected M+H, M+Na and M+K features per target compound across samples (corresponding to Table S1)
- Per-feature isotopic similarity scores against theoretical enviPat patterns
- Quantified isotopologue intensities per compound per sample

## How to apply

Define the target metabolite list (e.g. the 14 standard compounds of the MM14 centWave dataset) and compute their theoretical isotope patterns with enviPat based on the MS resolution. Load the centWave extracted ion chromatograms (EICs) and remove baselines with the airPLS method. Detect isotopologue peak positions with the MassSpecWavelet package, then align retention-time shifts across samples using the PAFFT method (Wong et al. 2005) so targeted isotopologues match across samples. Integrate and quantify the matched M+H, M+Na and M+K features and compute isotopic similarity scores against the theoretical patterns; run the whole pipeline reactively via the TarMet shiny GUI (runTarMet()) or via the R command-line vignette scripts.

## Related tools

- **TarMet** (Reactive shiny GUI application that orchestrates the entire targeted metabolite and isotopologue quantification workflow) — https://github.com/hcji/TarMet
- **shiny** (Provides TarMet's reactive graphical user interface) — https://github.com/hcji/TarMet
- **enviPat** (Calculates theoretical isotope patterns of target compounds based on MS resolution)
- **airPLS** (Removes baselines of all extracted ion chromatograms before peak detection)
- **MassSpecWavelet** (Detects peak positions of the isotopologues in the baseline-corrected EICs)
- **PAFFT** (Aligns retention-time shifts across samples to match targeted isotopologues (Wong et al. 2005))

## Examples

```
install_github("hcji/TarMet"); library(TarMet); runTarMet()
```

## Evaluation signals

- On the MM14 dataset, TarMet should detect 14 M+H, 13 M+Na and 8 M+K features for the 14 standard compounds.
- Average isotopic similarities should be approximately 0.95 (M+H), 0.93 (M+Na) and 0.90 (M+K), matching Table S1.
- Feature counts and similarity scores should exceed those of state-of-the-art comparison methods, as reported in the abstract.
- Every quantified feature should have an isotopic similarity score computed against the enviPat theoretical pattern at the specified MS resolution.
- Matched isotopologues should be retention-time aligned across samples via PAFFT before integration.

## Limitations

- Data-independent acquisition support is a planned future feature, not yet implemented.
- MS/MS library verification is a planned future feature, not yet implemented.
- Requires manually defined target metabolites; isotopologue detection quality depends on the MS-resolution-based theoretical patterns.

## Evidence
