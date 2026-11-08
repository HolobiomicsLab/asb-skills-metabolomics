---
name: ms2-peak-mass-decomposition
description: Use when annotating MS2 fragment peaks with chemical subformulae for
  a candidate precursor formula and you want to avoid costly fragmentation tree computation
  (e.g. as input features for formula scoring models like MIST-CF).
when_to_use_negative:
- When a full fragmentation tree with fragmentation-event edges is explicitly required
  (e.g. SIRIUS-style tree scoring) — this routine deliberately circumvents tree construction
- 'For negative-mode data or adduct-switching scenarios: MIST-CF models were trained
  only for positive-mode data and adduct switching in MS2 subformula assignment is
  not considered'
- When the candidate precursor formula is unknown — the routine enumerates subformulae
  of a given formula and cannot proceed without one
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Provides the dynamic-programming algorithm 'SIRIUS decomp' used to enumerate
    all potential formulae for an observed mass (subformula decomposition of fragment
    masses)
  repo: https://github.com/boecker-lab/sirius
- name: MIST-CF
  role: Consumes the peak-to-subformula assignment table as input features for energy-based
    chemical formula scoring
  repo: github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:5d6fe35d93edc0abe2ad21b25bcc04ae5868608ba18858d7f4f029b25deaf7f8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ms2-peak-mass-decomposition

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A lightweight MS2 peak subformula assignment routine from MIST-CF that replaces fragmentation tree construction: subtract the precursor adduct mass, enumerate candidate subformulae of the precursor formula, and match each peak to its nearest subformula within an instrument-specific ppm tolerance.

## When to use

Use when annotating MS2 fragment peaks with chemical subformulae for a candidate precursor formula and you want to avoid costly fragmentation tree computation (e.g. as input features for formula scoring models like MIST-CF).

## When NOT to use

- When a full fragmentation tree with fragmentation-event edges is explicitly required (e.g. SIRIUS-style tree scoring) — this routine deliberately circumvents tree construction
- For negative-mode data or adduct-switching scenarios: MIST-CF models were trained only for positive-mode data and adduct switching in MS2 subformula assignment is not considered
- When the candidate precursor formula is unknown — the routine enumerates subformulae of a given formula and cannot proceed without one

## Inputs

- MS2 peak list (e.g. peaks from an MGF entry such as data/demo_specs.mgf)
- Candidate precursor chemical formula and adduct (e.g. [M+H]+)
- Instrument type (Unknown/Ion Trap, Q-ToF, Orbitrap, or FTICR) determining the ppm tolerance

## Outputs

- assignments.tsv: one row per MS2 peak with its assigned subformula (within the ppm tolerance)
- Per-peak fragment and neutral loss formulae used as MIST-CF model features

## How to apply

Parse the MS2 peak list and the candidate chemical formula, then subtract the precursor adduct mass from each peak m/z to obtain fragment ion masses. Enumerate all possible subformulae of the candidate formula using SIRIUS's dynamic-programming decomposition ('SIRIUS decomp', set via the SIRIUS_PATH environment variable), which MIST-CF relies on for its fast implementation of exact-mass-to-formula decomposition. Filter the enumerated subformulae by requiring RDBE >= 0 to exclude chemically impossible fragments. For each peak, assign the subformula whose exact mass is nearest within the instrument-specific ppm tolerance: 15 ppm for Unknown/Ion Trap, 10 ppm for Q-ToF, and 5 ppm for Orbitrap/FTICR instruments. Write the peak-to-subformula assignments to assignments.tsv, embedding the neutral loss fragment formula for each peak in addition to the fragment formula.

## Related tools

- **SIRIUS** (Provides the dynamic-programming algorithm 'SIRIUS decomp' used to enumerate all potential formulae for an observed mass (subformula decomposition of fragment masses)) — https://github.com/boecker-lab/sirius
- **MIST-CF** (Consumes the peak-to-subformula assignment table as input features for energy-based chemical formula scoring) — github.com/samgoldman97/mist-cf

## Examples

```
SIRIUS_PATH=./sirius-5.5.7-linux64-headless/bin/sirius; python -m mist_cf.sh.mist_cf_ms2 --data data/demo_specs.mgf --out-dir quickstart/mist_cf_out/
```

## Evaluation signals

- assignments.tsv exists and contains one row per MS2 peak
- Every assigned subformula's exact mass matches the corresponding peak within the stated instrument-specific ppm tolerance (15 ppm Unknown/Ion Trap, 10 ppm Q-ToF, 5 ppm Orbitrap/FTICR)
- No assigned subformula has RDBE < 0
- Each peak's assigned fragment mass is consistent with a subformula of the candidate precursor formula after adduct mass subtraction

## Limitations

- Adduct switching in MS2 subformula assignment is not considered
- Models using these assignments were trained only for positive-mode data; negative-mode support is absent
- MIST-CF still relies upon SIRIUS's algorithmic decomposition of exact masses into formula candidates, so SIRIUS must be installed and SIRIUS_PATH set

## Evidence

- [methods] MIST-CF assigns each peak mass its nearest subformula match within an instrument-specific ppm tolerance: Unknown/Ion Trap 15 ppm, Q-ToF 10 ppm, Orbitrap/FTICR 5 ppm.: [evidence span withheld — anchor status: not_found]
