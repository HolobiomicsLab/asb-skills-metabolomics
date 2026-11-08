---
name: isotope-pattern-calculation
description: Use when quantifying targeted metabolites or isotope-tracer-labelled
  isotopologues in LC-MS data and a theoretical isotope envelope (e.g. up to six 13C
  labels) is needed to match and score detected peaks.
when_to_use_negative:
- The input is already a verified feature/peak table and no isotope-envelope matching
  is required.
- Data are untargeted or MS/MS-based rather than targeted LC-MS quantification (TarMet's
  README lists MS/MS library verification and data-independent acquisition as future,
  not current, features).
- No molecular formulas for the targets are available, since enviPat cannot compute
  patterns without them.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: enviPat
  role: Calculates the theoretical isotope patterns of the target compounds based
    on the resolution of the MS; the reference envelopes used for isotopologue matching
    and similarity scoring
  repo: https://github.com/hcji/TarMet
- name: TarMet
  role: Shiny application that invokes enviPat, airPLS, MassSpecWavelet and PAFFT
    to turn theoretical patterns into quantified isotopologue peak areas
  repo: https://github.com/hcji/TarMet
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:1d26b9fd6ec8560ba908f950a3cceffe23b98373d7d4c6ee1069bd750f40fee1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# isotope-pattern-calculation

## Summary

Calculate theoretical isotope patterns of target metabolites (or their 13C-labelled isotopologues) from molecular formulas using the enviPat package, at the resolution of the MS instrument, as the reference against which measured peaks are scored for isotopic similarity in TarMet.

## When to use

Use when quantifying targeted metabolites or isotope-tracer-labelled isotopologues in LC-MS data and a theoretical isotope envelope (e.g. up to six 13C labels) is needed to match and score detected peaks.

## When NOT to use

- The input is already a verified feature/peak table and no isotope-envelope matching is required.
- Data are untargeted or MS/MS-based rather than targeted LC-MS quantification (TarMet's README lists MS/MS library verification and data-independent acquisition as future, not current, features).
- No molecular formulas for the targets are available, since enviPat cannot compute patterns without them.

## Inputs

- Target metabolite list with molecular formulas (metabolites and adducts such as M+H, M+Na, M+K, or isotope-tracer variants)
- MS instrument resolution
- Maximum isotope label count (e.g. max 13C = 6 for 13C6-glucose tracer assays)

## Outputs

- Theoretical isotope patterns (isotope envelopes) per target metabolite and per isotopologue
- Reference patterns used to compute isotopic similarity scores and to assign detected peaks to M+H / M+Na / M+K or labelled isotopologues

## How to apply

Define the target metabolites (including adduct forms such as M+H, M+Na, M+K, or stable-isotope tracer products with up to the maximum 13C count) and compute their theoretical isotope patterns with the enviPat package, taking the resolution of the MS instrument as the input parameter so the simulated envelope matches the observed peak widths. The resulting per-compound isotope envelopes serve as the reference in TarMet's targeted workflow: extracted ion chromatograms (EICs) are baseline-corrected with airPLS, peak positions detected with MassSpecWavelet, and retention-time shifts aligned across samples with PAFFT before each detected isotopologue is matched against the theoretical pattern. In tracer mode (e.g. 13C6-glucose labelling of MRC5 fibroblast cellular metabolites), calculate patterns for every isotopologue of each target so peak areas of all labelled species can be integrated automatically. Correctness is judged downstream by the isotopic similarity scores between measured and theoretical patterns (average ~0.95 for M+H, ~0.93 for M+Na, ~0.90 for M+K features in the MM14 centWave dataset), so the theoretical patterns must be regenerated whenever the instrument resolution or target list changes.

## Related tools

- **enviPat** (Calculates the theoretical isotope patterns of the target compounds based on the resolution of the MS; the reference envelopes used for isotopologue matching and similarity scoring) — https://github.com/hcji/TarMet
- **TarMet** (Shiny application that invokes enviPat, airPLS, MassSpecWavelet and PAFFT to turn theoretical patterns into quantified isotopologue peak areas) — https://github.com/hcji/TarMet

## Examples

```
install.packages("enviPat"); library(TarMet); runTarMet()
```

## Evaluation signals

- Detected M+H / M+Na / M+K features of the 14 MM14 standard compounds should number 14 / 13 / 8 with average isotopic similarities of 0.95 / 0.93 / 0.90.
- Isotopic similarity between measured and theoretical patterns should be high (approaching 1) for correctly matched isotopologues.
- Theoretical patterns must be recomputed when the MS resolution or the target metabolite list changes.
- For tracer assays, all isotopologues up to the specified maximum 13C count should have computed patterns and quantified peak areas.
- Quantified isotopologue peak areas from parallel samples should show high consistency, and 60 min 13C6-glucose samples should show higher 13C incorporation than 5 min samples.

## Limitations

- Patterns depend on the stated MS resolution; mismatched resolution degrades similarity scores.
- Cannot be applied to data-independent acquisition or MS/MS library verification, which are planned future features of TarMet.
- Requires molecular formulas for every target metabolite; compounds without formulas cannot be handled.

## Evidence

- [results] Running TarMet on the MM14 dataset detects 14 M+H, 13 M+Na and 8 M+K features of the 14 compounds, with average isotopic similarities of 0.95, 0.93 and 0.90 respectively.: [evidence span withheld — anchor status: not_found]
- [methods] Calculate theoretical isotope patterns of the target metabolites for up to six 13C labels via the enviPat package based on the MS resolution.: [evidence span withheld — anchor status: not_found]
