---
name: ionization-mode-filtering
description: Use when assembling curated training libraries (library.csv) or benchmark
  query CSVs from NIST'17, MS-DIAL, or MSnLib spectra and entries carry heterogeneous
  precursor types that must be reduced to the modes used by the model.
when_to_use_negative:
- When the analysis targets precursor types outside [M+H]+, [M-H]-, [M]+, or [M]-
  (e.g. [M+Na]+ or [M+NH4]+ adducts), which FIORA's filtering explicitly excludes
- When the goal is multi-stage (MSn) fragmentation beyond MS/MS, which the single-step
  FIORA pipeline does not model
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Curation/training pipeline whose library loader applies the ionization-type
    filter, and whose fiora-train CLI accepts a --precursor-modes argument restricting
    training modes
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:a635e4ac075cd2055c9995a3796a6eaad9b198dee00a1e595d651a023d0dce35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# ionization-mode-filtering

## Summary

Restrict spectral library entries and benchmark queries to supported precursor/ionization types — [M+H]+ and [M-H]-, extended with [M]+/[M]- for MSnLib — so that downstream FIORA training, curation, and evaluation operate only on adduct modes the model actually handles.

## When to use

Use when assembling curated training libraries (library.csv) or benchmark query CSVs from NIST'17, MS-DIAL, or MSnLib spectra and entries carry heterogeneous precursor types that must be reduced to the modes used by the model.

## When NOT to use

- When the analysis targets precursor types outside [M+H]+, [M-H]-, [M]+, or [M]- (e.g. [M+Na]+ or [M+NH4]+ adducts), which FIORA's filtering explicitly excludes
- When the goal is multi-stage (MSn) fragmentation beyond MS/MS, which the single-step FIORA pipeline does not model

## Inputs

- Raw library spectra parsed from NIST'17 (.msp-style entries), MS-DIAL .msp files, and MSnLib *_ms2.mgf files
- Benchmark query CSVs with columns Name, SMILES, Precursor_type, CE, Instrument_type

## Outputs

- Filtered spectral entries (library.csv) containing only [M+H]+, [M-H]- (plus [M]+/[M]- for MSnLib) precursors passing the 1000 Da / 100 eV limits
- Mode-restricted benchmark query CSVs per test split and ionization mode

## How to apply

After parsing raw NIST'17, MS-DIAL, and MSnLib library entries and converting NCE collision energies to eV, drop every spectrum whose precursor/ionization type is not [M+H]+ or [M-H]-; for MSnLib additionally retain [M]+ and [M]- ions. Apply this adduct filter alongside the structural consistency checks (SMILES/MOL/InChIKey) and the hard limits of maximum compound weight 1000 Da and maximum collision energy 100 eV. The same mode restriction is propagated into benchmark query CSVs (Name, SMILES, Precursor_type, CE, Instrument_type) so that fiora-predict only receives supported precursor types, and into fiora-train via the --precursor-modes argument when retraining on MSnLib.

## Related tools

- **FIORA** (Curation/training pipeline whose library loader applies the ionization-type filter, and whose fiora-train CLI accepts a --precursor-modes argument restricting training modes) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-train -i resources/data/msnlib/library.csv -o checkpoints/fiora.pt --device cuda:0 --instruments HCD --precursor-modes "[M+H]+,[M-H]-,[M]+,[M]-"
```

## Evaluation signals

- Every retained entry in library.csv has Precursor_type in {[M+H]+, [M-H]-} (plus {[M]+, [M]-} for MSnLib-derived entries); any other adduct string indicates a filter failure
- Cross-check that filtered entries also satisfy the companion hard limits (compound weight <= 1000 Da, collision energy <= 100 eV) and structural consistency checks
- Benchmark query CSVs per ionization mode contain no unsupported precursor types, matching the modes evaluated in Table 1 comparisons
- fiora-train run logs confirm the --precursor-modes list matches the intended mode set for the training data

## Limitations

- Restricting to [M+H]+ and [M-H]- (plus [M]+/[M]- for MSnLib) discards all other adduct spectra, shrinking usable training and evaluation data
- The adduct filter alone does not guarantee quality; it must be combined with the >=2 peak-match, >=50% intensity-coverage, and <=90% precursor-intensity retention filters
- MSnLib-specific allowance of [M]+/[M]- means the same filter rule set is not uniform across library sources

## Evidence

- [discussion] Ionization-type filter scope: "we filtered for ionization types [M+H]+ and [M-H]- (in case of MSnLib [M]+ and [M]- ions were considered as well)"
- [discussion] Companion hard limits applied with the adduct filter: "These requirements included a maximum weight of 1000 Da and maximum collision energy of 100 eV."
- [readme] README CLI flag exposing the mode filter for training: "--precursor-modes "[M+H]+,[M-H]-,[M]+,[M]-""
