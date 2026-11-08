---
name: isotopologue-quantification
description: Use when centWave-processed LC-MS chromatograms of known target metabolites
  must be quantified per adduct (M+H, M+Na, M+K) with isotopologue-level confidence,
  including isotope tracer assays.
when_to_use_negative:
- Input is already a feature table or peak matrix — TarMet expects raw centWave EIC
  chromatograms to re-detect isotopologues.
- Untargeted metabolite discovery where no predefined target metabolite list exists
  — the workflow requires theoretical isotope patterns of known targets.
- Data-independent acquisition or workflows requiring MS/MS library verification —
  listed as future features, not supported.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: TarMet
  role: Shiny application implementing the full isotopologue detection, alignment,
    quantification and similarity-scoring workflow
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Calculates theoretical isotope patterns of target compounds from the MS resolution
- name: airPLS
  role: Removes baselines of extracted ion chromatograms before peak detection
- name: MassSpecWavelet
  role: Detects isotopologue peak positions in EICs
- name: PAFFT
  role: Aligns retention-time shifts across samples to match targeted isotopologues
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:e2bef3ad3f281cb2a4346a4b5ef07d6205bbdd38cc7e917ed7c0bfd91ab1a0e6
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# isotopologue-quantification

## Summary

Targeted detection and quantification of metabolite isotopologues (e.g., M+H, M+Na, M+K adducts) from LC-MS chromatograms, scoring each feature against theoretical isotope patterns. Implemented in the TarMet shiny application for targeted metabolic and stable isotope tracer analysis.

## When to use

Use when centWave-processed LC-MS chromatograms of known target metabolites must be quantified per adduct (M+H, M+Na, M+K) with isotopologue-level confidence, including isotope tracer assays.

## When NOT to use

- Input is already a feature table or peak matrix — TarMet expects raw centWave EIC chromatograms to re-detect isotopologues.
- Untargeted metabolite discovery where no predefined target metabolite list exists — the workflow requires theoretical isotope patterns of known targets.
- Data-independent acquisition or workflows requiring MS/MS library verification — listed as future features, not supported.

## Inputs

- centWave-processed LC-MS chromatograms / extracted ion chromatograms (e.g., MM14 dataset)
- List of target metabolite definitions (structures/formulas for isotope-pattern calculation)
- MS resolution (for theoretical isotope pattern generation)

## Outputs

- Peak table of detected M+H, M+Na, M+K features per target compound (Table S1-style counts)
- Quantified isotopologue intensities per sample
- Isotopic similarity scores of detected features against theoretical isotope patterns

## How to apply

Define the target metabolites (e.g., the 14 standard compounds of the MM14 dataset) and compute their theoretical isotope patterns at the MS resolution using enviPat. Load centWave extracted ion chromatograms (MM14 available at http://msbi.ipb-halle.de/msbi/centwave), remove EIC baselines with airPLS, and detect isotopologue peak positions with MassSpecWavelet. Align retention-time shifts across samples with the PAFFT method (Wong et al. 2005) so targeted isotopologues match across samples. Integrate and quantify the matched M+H, M+Na and M+K features and compute isotopic similarity scores against the theoretical patterns; aggregate per-compound counts and average similarities into a peak table (corresponding to Table S1). Decision point: if similarity scores or detected counts fall below benchmark values (MM14: 14 M+H, 13 M+Na, 8 M+K features; average similarities 0.95/0.93/0.90), revisit baseline correction or alignment parameters.

## Related tools

- **TarMet** (Shiny application implementing the full isotopologue detection, alignment, quantification and similarity-scoring workflow) — https://github.com/hcji/TarMet
- **enviPat** (Calculates theoretical isotope patterns of target compounds from the MS resolution)
- **airPLS** (Removes baselines of extracted ion chromatograms before peak detection)
- **MassSpecWavelet** (Detects isotopologue peak positions in EICs)
- **PAFFT** (Aligns retention-time shifts across samples to match targeted isotopologues)

## Examples

```
library(devtools); install_github("hcji/TarMet"); library(TarMet); runTarMet()
```

## Evaluation signals

- On the MM14 benchmark, TarMet should detect 14 M+H, 13 M+Na and 8 M+K features of the 14 standard compounds.
- Average isotopic similarities for the MM14 dataset should be approximately 0.95 (M+H), 0.93 (M+Na) and 0.90 (M+K).
- Per-compound counts and similarity scores should reproduce the values reported in Table S1 of the paper.
- Detected isotopologue counts and quantification quality should equal or exceed state-of-the-art methods (e.g., AssayR on the cellular metabolites validation dataset).

## Limitations

- Does not yet support data-independent acquisition (planned future feature).
- Does not yet support MS/MS library verification (planned future feature).
- Quantification depends on accurate RT alignment via PAFFT; large retention-time shifts across samples may degrade isotopologue matching.
- Requires predefined target metabolites — untargeted screening is out of scope.

## Evidence
