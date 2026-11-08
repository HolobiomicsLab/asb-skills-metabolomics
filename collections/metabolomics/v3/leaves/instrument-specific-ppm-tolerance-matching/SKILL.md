---
name: instrument-specific-ppm-tolerance-matching
description: Use when annotating MS2 fragment peaks with subformulae of a candidate
  precursor formula and no fragmentation tree is desired; instrument type (Ion Trap,
  Q-ToF, Orbitrap/FTICR) determines the ppm tolerance.
when_to_use_negative:
- Negative-mode MS/MS data — MIST-CF models were trained only for positive-mode data
- When adduct switching between MS1 precursor and MS2 fragments must be modeled —
  the authors explicitly do not consider adduct switching in MS2 subformula assignment
- When you need isotope-pattern or MS1 isotopic scoring integrated with subformula
  matching — this integration is not addressed
tools:
- name: MIST-CF
  role: Implements the internal chemical subformula assignment protocol with instrument-specific
    ppm tolerances and writes assignments.tsv
  repo: github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the fast dynamic-programming algorithmic decomposition (SIRIUS decomp)
    of exact fragment masses into candidate subformulae
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:5c74929ea50b8d0b9726de6370aaef2df11dea2b8253c16c9344df035a1726f7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# instrument-specific-ppm-tolerance-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Assigns each MS2 peak (after precursor/adduct mass subtraction) its nearest candidate chemical subformula within an instrument-specific mass tolerance: 15 ppm for Unknown/Ion Trap, 10 ppm for Q-ToF, and 5 ppm for Orbitrap/FTICR. This is MIST-CF's simple peak subformula assignment routine that replaces fragmentation tree construction.

## When to use

Use when annotating MS2 fragment peaks with subformulae of a candidate precursor formula and no fragmentation tree is desired; instrument type (Ion Trap, Q-ToF, Orbitrap/FTICR) determines the ppm tolerance.

## When NOT to use

- Negative-mode MS/MS data — MIST-CF models were trained only for positive-mode data
- When adduct switching between MS1 precursor and MS2 fragments must be modeled — the authors explicitly do not consider adduct switching in MS2 subformula assignment
- When you need isotope-pattern or MS1 isotopic scoring integrated with subformula matching — this integration is not addressed

## Inputs

- MS2 peak list (e.g. spectra in data/demo_specs.mgf)
- Candidate chemical formula / precursor formula with adduct (e.g. [M+H]+ and other positive-mode adducts)
- Instrument type annotation (Unknown/Ion Trap, Q-ToF, or Orbitrap/FTICR)

## Outputs

- assignments.tsv: peak-to-subformula assignment table with one row per MS2 peak

## How to apply

First subtract the precursor adduct mass from each MS2 peak m/z to obtain fragment ion masses, then enumerate candidate subformulae of the candidate chemical formula using SIRIUS's fast dynamic-programming decomposition (SIRIUS decomp). Filter subformulae by requiring RDBE >= 0 to exclude chemically impossible fragments. For each peak, select the subformula whose exact mass is nearest within the instrument-specific tolerance — 15 ppm for Unknown/Ion Trap, 10 ppm for Q-ToF, and 5 ppm for Orbitrap/FTICR — since higher-resolution instruments justify tighter matching. MIST-CF implements this internal subformula assignment protocol rather than SIRIUS fragmentation trees, and the tool embeds the instrument type as a model covariate. Write the peak-to-subformula assignment table as assignments.tsv, one row per MS2 peak.

## Related tools

- **MIST-CF** (Implements the internal chemical subformula assignment protocol with instrument-specific ppm tolerances and writes assignments.tsv) — github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the fast dynamic-programming algorithmic decomposition (SIRIUS decomp) of exact fragment masses into candidate subformulae) — https://github.com/boecker-lab/sirius

## Examples

```
. quickstart/run_model.sh
```

## Evaluation signals

- assignments.tsv exists and contains one row per MS2 peak
- Every assigned subformula mass matches its corresponding peak within the stated ppm tolerance (15/10/5 ppm depending on instrument)
- All assigned subformulae pass the RDBE >= 0 filter (no chemically impossible fragments assigned)
- Tolerance selection matches the declared instrument type of the spectrum (Ion Trap/Unknown 15 ppm, Q-ToF 10 ppm, Orbitrap/FTICR 5 ppm)

## Limitations

- Tolerance is fixed per instrument class; only Unknown/Ion Trap (15 ppm), Q-ToF (10 ppm), and Orbitrap/FTICR (5 ppm) are defined
- Adduct switching in MS2 subformula assignment is not considered
- Subformula candidate enumeration still depends on SIRIUS's algorithmic decomposition implementation
- Positive-mode only; no negative-mode support

## Evidence

- [full_text] MIST-CF assigns each peak mass its nearest subformula match within an instrument-specific ppm tolerance: Unknown/Ion Trap 15 ppm, Q-ToF 10 ppm, Orbitrap/FTICR 5 ppm.: [evidence span withheld — anchor status: not_found]
