---
name: subformula-enumeration-from-precursor-formula
description: Use when annotating MS2 peaks from a tandem mass spectrum against a candidate
  molecular formula, e.g. de novo formula ranking with MIST-CF, and no fragmentation
  tree is required.
when_to_use_negative:
- Spectra acquired in negative ion mode — MIST-CF models were trained only on positive-mode
  data
- Analyses that require true fragmentation trees with fragmentation-event edges and
  neutral-loss tree structure — this protocol only does independent peak-wise subformula
  matching
- Cases involving adduct switching in MS2 subformula assignment, which the MIST-CF
  authors explicitly do not consider
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Provides the `SIRIUS decomp` dynamic programming module used to enumerate
    all potential subformulae of a candidate mass/formula
  repo: https://github.com/boecker-lab/sirius
- name: MIST-CF
  role: Runs the internal peak subformula assignment protocol (adduct-mass subtraction,
    RDBE filter, ppm-bounded nearest match) in place of SIRIUS fragmentation trees
  repo: github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:82a0d567eaa8a7df25f7e831c19ef084d18da2616ee7d493e55815124770eee7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Subformula enumeration from precursor formula

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Assigns MS2 fragment peaks to chemical subformulae of a candidate precursor formula by enumerating candidate formulae (via SIRIUS decomp), filtering with an RDBE >= 0 heuristic, and matching each peak to its nearest subformula mass within an instrument-specific ppm tolerance — replacing fragmentation tree construction in MIST-CF.

## When to use

Use when annotating MS2 peaks from a tandem mass spectrum against a candidate molecular formula, e.g. de novo formula ranking with MIST-CF, and no fragmentation tree is required.

## When NOT to use

- Spectra acquired in negative ion mode — MIST-CF models were trained only on positive-mode data
- Analyses that require true fragmentation trees with fragmentation-event edges and neutral-loss tree structure — this protocol only does independent peak-wise subformula matching
- Cases involving adduct switching in MS2 subformula assignment, which the MIST-CF authors explicitly do not consider

## Inputs

- MS2 peak list (e.g. MGF spectra such as data/demo_specs.mgf)
- Candidate chemical formula (precursor formula candidate, optionally with adduct type)
- Instrument type (Unknown/Ion Trap, Q-ToF, or Orbitrap/FTICR)

## Outputs

- assignments.tsv: peak-to-subformula assignment table with one row per MS2 peak

## How to apply

Parse the MS2 peak list and the candidate chemical formula, then subtract the precursor adduct mass from each peak m/z to obtain fragment ion masses. Enumerate all subformulae of the candidate formula using SIRIUS's dynamic programming decomposition module (`SIRIUS decomp`), which MIST-CF uses for its fast implementation. Filter candidates by requiring RDBE >= 0 to exclude chemically impossible fragments. For each peak, select the subformula whose exact mass is nearest within the instrument-specific tolerance: 15 ppm for Unknown/Ion Trap, 10 ppm for Q-ToF, and 5 ppm for Orbitrap/FTICR instruments. Write the peak-to-subformula assignments to assignments.tsv, which feeds the downstream MIST-CF spectrum-transformer scoring instead of a SIRIUS fragmentation tree.

## Related tools

- **SIRIUS** (Provides the `SIRIUS decomp` dynamic programming module used to enumerate all potential subformulae of a candidate mass/formula) — https://github.com/boecker-lab/sirius
- **MIST-CF** (Runs the internal peak subformula assignment protocol (adduct-mass subtraction, RDBE filter, ppm-bounded nearest match) in place of SIRIUS fragmentation trees) — github.com/samgoldman97/mist-cf

## Examples

```
export SIRIUS_PATH=${your_sirius_path}; . quickstart/run_model.sh  # annotates data/demo_specs.mgf via the subformula assignment protocol
```

## Evaluation signals

- assignments.tsv exists and contains one row per MS2 peak in the input spectrum
- Every assigned subformula exact mass matches its corresponding peak within the stated ppm tolerance (15 ppm Unknown/Ion Trap, 10 ppm Q-ToF, 5 ppm Orbitrap/FTICR)
- All retained subformulae have RDBE >= 0 (no chemically impossible fragments assigned)
- Each assigned subformula is a valid decomposition of the candidate precursor formula after adduct mass subtraction
- On the CASMI2022 benchmark the full MIST-CF pipeline using these assignments ties the winning positive-mode SIRIUS solution without MS1 isotopic information

## Limitations

- PPM tolerances are instrument-specific; using the wrong tolerance for the instrument type (e.g. 15 ppm on Orbitrap data) will produce incorrect assignments
- The authors rely on SIRIUS for the algorithmic decomposition of exact masses into formula candidates, so the step inherits that dependency and its element-constraint settings
- Trained models and evaluation were positive-mode only; NIST20-trained models (better for Orbitrap/high-resolution data) are available only upon reasonable request to licensed users

## Evidence
