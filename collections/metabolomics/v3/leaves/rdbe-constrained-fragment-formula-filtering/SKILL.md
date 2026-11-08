---
name: rdbe-constrained-fragment-formula-filtering
description: Use when annotating MS2 fragment peaks with chemical subformulae for
  a candidate molecular formula (e.g., MIST-CF pipeline) and you need a fast, tree-free
  peak-to-formula assignment instead of SIRIUS fragmentation tree computation.
when_to_use_negative:
- Negative-mode MS/MS data — MIST-CF models (and the subformula assignment step they
  consume) were trained only for positive-mode data
- Analyses that require explicit fragmentation-event relationships (arcs/losses between
  fragment formulae) rather than independent per-peak subformula labels — this routine
  deliberately avoids fragmentation tree construction
- Cases where the fragment subformulae may come from a different adduct than the precursor
  — the article notes 'we do not consider adduct switching in MS2 subformula assignment'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MIST-CF
  role: Pipeline whose internal subformula assignment protocol this skill implements;
    writes assignments.tsv feeding the formula transformer
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the fast dynamic-programming decomposition (SIRIUS decomp) used to
    enumerate all candidate subformulae for each exact fragment mass
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:def57d95cb22b681803940d541258719c9b9d7ba8ca61282e138ce3eb68bf7f0
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# RDBE-constrained fragment subformula assignment with instrument-specific ppm tolerances

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

MIST-CF's internal subformula assignment step: after subtracting the precursor adduct mass from MS2 peak masses, candidate subformulae are enumerated via SIRIUS-style decomposition, filtered with an RDBE >= 0 heuristic, and each peak is matched to its nearest subformula within an instrument-specific ppm tolerance — replacing fragmentation tree construction.

## When to use

Use when annotating MS2 fragment peaks with chemical subformulae for a candidate molecular formula (e.g., MIST-CF pipeline) and you need a fast, tree-free peak-to-formula assignment instead of SIRIUS fragmentation tree computation.

## When NOT to use

- Negative-mode MS/MS data — MIST-CF models (and the subformula assignment step they consume) were trained only for positive-mode data
- Analyses that require explicit fragmentation-event relationships (arcs/losses between fragment formulae) rather than independent per-peak subformula labels — this routine deliberately avoids fragmentation tree construction
- Cases where the fragment subformulae may come from a different adduct than the precursor — the article notes 'we do not consider adduct switching in MS2 subformula assignment'

## Inputs

- MS2 peak list (e.g., MGF spectra such as data/demo_specs.mgf)
- Candidate chemical formula for the precursor (user-defined or from a prior candidate-ranking step)
- Precursor adduct type (e.g., [M+H]+; positive mode only)
- Instrument type (Unknown/Ion Trap, Q-ToF, or Orbitrap/FTICR) to select the ppm tolerance
- SIRIUS 5.x installation (SIRIUS decomp module) for formula enumeration

## Outputs

- assignments.tsv: one row per MS2 peak with its assigned subformula
- Per-peak subformula assignments used as input features to the MIST-CF formula transformer (fragment and neutral-loss formula embeddings)

## How to apply

Parse the input MS2 peak list (e.g., demo_specs.mgf) together with the candidate chemical formula, and subtract the precursor adduct mass from each peak m/z to obtain fragment ion masses. Enumerate all possible subformulae of the candidate formula using SIRIUS's fast dynamic-programming exact-mass decomposition (SIRIUS decomp, via the SIRIUS_PATH environment variable). Filter candidates with the RDBE (ring double bond equivalent) heuristic, keeping only subformulae with RDBE >= 0 to exclude chemically impossible fragments. For each peak, select the subformula whose exact mass is nearest within the instrument-specific tolerance: 15 ppm for Unknown/Ion Trap, 10 ppm for Q-ToF, and 5 ppm for Orbitrap/FTICR instruments. Write the resulting peak-to-subformula mapping as assignments.tsv; this annotation protocol is what allows MIST-CF to circumvent fragmentation tree construction entirely.

## Related tools

- **MIST-CF** (Pipeline whose internal subformula assignment protocol this skill implements; writes assignments.tsv feeding the formula transformer) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the fast dynamic-programming decomposition (SIRIUS decomp) used to enumerate all candidate subformulae for each exact fragment mass) — https://github.com/boecker-lab/sirius

## Examples

```
wget https://github.com/boecker-lab/sirius/releases/download/v5.5.7/sirius-5.5.7-linux64-headless.zip && unzip sirius-5.5.7-linux64-headless.zip && export SIRIUS_PATH=./sirius-linux64-headless/sirius
```

## Evaluation signals

- assignments.tsv exists and contains one row per MS2 peak in the input spectrum
- Every assigned subformula's exact mass matches its peak within the stated instrument tolerance (5 ppm Orbitrap/FTICR, 10 ppm Q-ToF, 15 ppm Unknown/Ion Trap)
- All retained subformulae satisfy the RDBE >= 0 constraint; no chemically impossible fragments are emitted
- Each peak is matched to the nearest-mass subformula among the enumerated candidates, not an arbitrary one
- Fragment subformulae are consistent with being subformulae of the precursor formula after adduct-mass subtraction

## Limitations

- Adduct switching in MS2 subformula assignment is not considered; fragments are assumed to derive from the precursor adduct mass
- The method relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates due to its fast implementation, so SIRIUS must be installed (SIRIUS_PATH set)
- The RDBE >= 0 heuristic may exclude unusual but real fragment compositions, and the ppm tolerances are coarse: low-resolution (Ion Trap/Unknown) data get a loose 15 ppm window, increasing false nearest matches

## Evidence

- [discussion] Reliance on SIRIUS decomposition: [evidence span withheld — anchor status: not_found]
