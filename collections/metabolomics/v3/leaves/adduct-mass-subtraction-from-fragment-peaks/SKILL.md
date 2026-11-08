---
name: adduct-mass-subtraction-from-fragment-peaks
description: Use when an MS2 peak list must be annotated with subformulae of a candidate
  precursor formula to score formula/adduct candidates de novo — e.g. when building
  inputs for a formula transformer and fragmentation trees are too slow or unnecessary.
when_to_use_negative:
- 'Negative-mode spectra: MIST-CF models were trained only for positive-mode data,
  so adduct subtraction/annotation in negative ion mode is unsupported.'
- Analyses where adduct switching within MS2 fragments must be modeled — the paper
  explicitly does not consider adduct switching in MS2 subformula assignment.
- When a full fragmentation tree (fragment/loss relationships) is required rather
  than independent peak-level subformula labels — this protocol deliberately assigns
  peaks without tree construction.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: MIST-CF
  role: 'Implements the internal chemical subformula assignment protocol: adduct mass
    subtraction, subformula enumeration/filtering, and nearest-mass peak annotation
    at instrument-specific ppm tolerances'
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the fast algorithmic decomposition (SIRIUS decomp) of exact fragment
    masses into candidate subformulae, downloaded as a headless release and located
    via the SIRIUS_PATH environment variable
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:1a32e563cb70a1d1c70fc5191d3f779d1ff0645bd5154697d4b693e7846d2510
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# adduct-mass-subtraction-from-fragment-peaks

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Converts MS2 fragment peak m/z values into neutral fragment ion masses by subtracting the precursor adduct mass, then annotates each peak with its nearest subformula of the candidate chemical formula within an instrument-specific ppm tolerance. This is MIST-CF's internal subformula assignment protocol that replaces SIRIUS fragmentation tree construction.

## When to use

Use when an MS2 peak list must be annotated with subformulae of a candidate precursor formula to score formula/adduct candidates de novo — e.g. when building inputs for a formula transformer and fragmentation trees are too slow or unnecessary.

## When NOT to use

- Negative-mode spectra: MIST-CF models were trained only for positive-mode data, so adduct subtraction/annotation in negative ion mode is unsupported.
- Analyses where adduct switching within MS2 fragments must be modeled — the paper explicitly does not consider adduct switching in MS2 subformula assignment.
- When a full fragmentation tree (fragment/loss relationships) is required rather than independent peak-level subformula labels — this protocol deliberately assigns peaks without tree construction.

## Inputs

- MS2 peak list (e.g. spectra in an MGF file such as data/demo_specs.mgf)
- Candidate chemical formula for the precursor
- Precursor adduct type/mass (e.g. [M+H]+ and other positive-mode adducts)
- Instrument type (Unknown/Ion Trap, Q-ToF, or Orbitrap/FTICR), which sets the ppm tolerance

## Outputs

- assignments.tsv: peak-to-subformula assignment table with one row per MS2 peak

## How to apply

Parse the MS2 peak list (e.g. from an MGF like data/demo_specs.mgf) together with the candidate chemical formula and its adduct. Subtract the precursor adduct mass from each MS2 peak m/z to obtain neutral fragment ion masses. Enumerate candidate subformulae of the precursor formula using SIRIUS's dynamic-programming decomposition (SIRIUS decomp, invoked via the SIRIUS_PATH environment variable, e.g. version 5.5.7 headless). Filter candidates by requiring RDBE (ring double bond equivalent) >= 0 to exclude chemically impossible fragments. Assign each peak the subformula whose exact mass is nearest within the instrument-specific tolerance — 15 ppm for Unknown/Ion Trap, 10 ppm for Q-ToF, and 5 ppm for Orbitrap/FTICR — and write the peak-to-subformula table as assignments.tsv.

## Related tools

- **MIST-CF** (Implements the internal chemical subformula assignment protocol: adduct mass subtraction, subformula enumeration/filtering, and nearest-mass peak annotation at instrument-specific ppm tolerances) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the fast algorithmic decomposition (SIRIUS decomp) of exact fragment masses into candidate subformulae, downloaded as a headless release and located via the SIRIUS_PATH environment variable) — https://github.com/boecker-lab/sirius

## Examples

```
. quickstart/download_model.sh; . quickstart/run_model.sh
```

## Evaluation signals

- assignments.tsv exists after the run and contains exactly one row per MS2 peak in the input list.
- Every assigned subformula's exact mass matches its corresponding peak within the stated instrument-specific ppm tolerance (15 ppm Unknown/Ion Trap, 10 ppm Q-ToF, 5 ppm Orbitrap/FTICR).
- No assigned subformula has RDBE < 0, confirming the chemically impossible fragment filter was applied.
- Neutral fragment masses equal input peak m/z minus the precursor adduct mass for every row (arithmetic invariant).
- For a demo run, predictions on the 10 CASMI22 spectra in data/demo_specs.mgf land in quickstart/mist_cf_out/ without errors.

## Limitations

- Reliance on SIRIUS remains for the algorithmic decomposition of exact masses into formula candidates, even though fragmentation trees are avoided.
- Only positive-mode data is supported; models were trained exclusively on positive-mode spectra.
- Adduct switching in MS2 subformula assignment is not considered.
- The quickstart model (trained on public NPLIB1/CANOPUS data) may be less performant than NIST20-trained models, particularly for Orbitrap or higher-resolution data.

## Evidence

- [intro] circumvent fragmentation trees for peak annotation: "As part of this, we develop a simple peak subformula assignment routine, thus circumventing the need for fragmentation tree construction."
- [discussion] SIRIUS still used for mass decomposition: [evidence span withheld — anchor status: not_found]
- [discussion] no adduct switching in MS2 assignment: "we do not consider adduct switching in MS2 subformula assignment"
- [readme] internal subformula protocol replaces SIRIUS fragmentation trees: "Utilizing an internal chemical subformula assignment protocol (rather than SIRIUS fragmentation trees)"
- [readme] multiple positive-mode adducts considered: "Considering multiple adduct types beyond [M+H]+ (still only positive mode)"
