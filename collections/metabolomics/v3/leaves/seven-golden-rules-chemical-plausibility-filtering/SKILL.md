---
name: seven-golden-rules-chemical-plausibility-filtering
description: Use when an electronically denoised MS/MS spectrum (peak list + SMILES/formula
  + adduct) still contains spurious fragment ions, and you must decide which fragments
  are chemical noise by testing subformula losses against Seven Golden Rules plausibility
  heuristics.
when_to_use_negative:
- The input is a raw, electronically noisy spectrum — apply electronic_denoising first
  (removing ions with identical intensities > 4) before chemical plausibility filtering
- The molecular formula/SMILES of the precursor is unknown and cannot be computed
  — the master formula is a prerequisite for enumerating subformula losses
- The goal is to distinguish isomeric compounds or in-source fragments — the paper
  notes these cannot be differentiated when solely relying on MS/MS spectral matching,
  even after noise removal
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectral_denoising
  role: Python package whose formula_denoising function implements the chemical denoising
    workflow (prep_formula, get_pmz_statistics, slice_spectrum, get_all_subformulas,
    check_candidates/get_denoise_tag, add_spectra)
  repo: github.com/FanzhouKong/spectral_denoising
- name: seven_golden_rules
  role: check_ratio heuristic used to discard chemically impossible loss subformulas
    (e.g. CH12), with LEWIS and SENIOR checks ignored
  repo: github.com/FanzhouKong/spectral_denoising
- name: chemparse
  role: Parses the master formula into element counts when enumerating candidate subformula
    losses
- name: molmass
  role: Computes exact masses of candidate subformulas, enabling mass-sorted candidate
    search
- name: rdkit
  role: Derives molecular information (e.g. benzene substructure detection for [M+N2+H2O]
    reaction handling) from the input SMILES; requires Python 3.8–3.12
  repo: github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:955a073b955589692bfbd63c09a7af095ca9f4a0bf6159879ba7d4552aa59939
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - chemparse
  - molmass
  - numpy
  - rdkit
  - seven_golden_rules
  - spectral_denoising
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# seven-golden-rules-chemical-plausibility-filtering

## Summary

Chemical denoising step of Spectral Denoising that removes chemical noise ions from an MS/MS spectrum by retaining only fragments whose mass can be explained by a chemically plausible subformula loss from the precursor, validated with the Seven Golden Rules algorithm.

## When to use

Use when an electronically denoised MS/MS spectrum (peak list + SMILES/formula + adduct) still contains spurious fragment ions, and you must decide which fragments are chemical noise by testing subformula losses against Seven Golden Rules plausibility heuristics.

## When NOT to use

- The input is a raw, electronically noisy spectrum — apply electronic_denoising first (removing ions with identical intensities > 4) before chemical plausibility filtering
- The molecular formula/SMILES of the precursor is unknown and cannot be computed — the master formula is a prerequisite for enumerating subformula losses
- The goal is to distinguish isomeric compounds or in-source fragments — the paper notes these cannot be differentiated when solely relying on MS/MS spectral matching, even after noise removal

## Inputs

- Electronically denoised MS/MS peak list as a numpy array of [mass, intensity] pairs
- Molecular structure as SMILES string or molecular formula (e.g. 'C5H4N4O')
- Precursor adduct string (e.g. '[M+Na]+')
- Precursor m/z; mass_tolerance starting at 0.005 Da, auto-inflated up to a ±10 mDa maximum search window

## Outputs

- Chemically denoised spectrum array (e.g. saved as formula_denoised_spectrum.npy) containing only ions with a plausible subformula loss plus the re-attached precursor region
- Per-fragment denoise tags ('True'/'False') indicating whether each ion has a chemically plausible subformula loss

## How to apply

Derive the master precursor formula from the SMILES and adduct via prep_formula (e.g. SMILES 'O=c1nc[nH]c2nc[nH]c12' with adduct '[M+Na]+' yields 'C5H4N4NaO', plus N2/O when a benzene substructure is present to account for [M+N2+H2O] reactions). Estimate precursor statistics with get_pmz_statistics using an initial mass_tolerance of 0.005 Da (auto-inflated if observed precursor error exceeds it, capped at ±10 mDa), then slice the spectrum with slice_spectrum and denoise only the fragment region. Enumerate candidate subformulas and exact masses from the master formula with get_all_subformulas, sorted by mass to accelerate search. For each fragment ion, search for a plausible subformula loss via check_candidates/get_denoise_tag, validating candidate losses with the seven_golden_rules.check_ratio heuristic (rejecting impossible compositions such as CH12) while deliberately ignoring the LEWIS and SENIOR checks designed for intact molecules. Retain only ions tagged 'True', re-attach the intact precursor region with add_spectra, and save the result (e.g. formula_denoised_spectrum.npy).

## Related tools

- **spectral_denoising** (Python package whose formula_denoising function implements the chemical denoising workflow (prep_formula, get_pmz_statistics, slice_spectrum, get_all_subformulas, check_candidates/get_denoise_tag, add_spectra)) — github.com/FanzhouKong/spectral_denoising
- **seven_golden_rules** (check_ratio heuristic used to discard chemically impossible loss subformulas (e.g. CH12), with LEWIS and SENIOR checks ignored) — github.com/FanzhouKong/spectral_denoising
- **chemparse** (Parses the master formula into element counts when enumerating candidate subformula losses)
- **molmass** (Computes exact masses of candidate subformulas, enabling mass-sorted candidate search)
- **rdkit** (Derives molecular information (e.g. benzene substructure detection for [M+N2+H2O] reaction handling) from the input SMILES; requires Python 3.8–3.12) — github.com/FanzhouKong/spectral_denoising

## Examples

```
peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)
```

## Evaluation signals

- Entropy similarity of the chemically denoised spectrum against the clean reference reaches ~1.00 (the demo reports 0.33 with chemical noise vs 1.00 after denoising)
- Chemically impossible loss compositions (e.g. CH12) are rejected by the seven_golden_rules.check_ratio heuristic
- LEWIS and SENIOR checks are NOT applied — losses are judged with the remaining Seven Golden Rules heuristics only
- The intact precursor region is preserved in the output spectrum after add_spectra re-attachment
- Candidate subformulas are enumerated from the master formula (including N2/O additions for substituted aromatics) and sorted by mass before fragment search

## Limitations

- Chemical plausibility checks are limited to the Seven Golden Rules algorithms without considering molecular connectivity, so not all artificially added noise ions are removed
- LEWIS and SENIOR checks are intentionally ignored, so valence/connectivity validity of losses is not fully enforced
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives

## Evidence

- [results] Seven Golden Rules validation with LEWIS/SENIOR ignored: "validated using the Seven Golden Rules algorithm to discard chemically impossible losses (for example, CH12)32 while ignoring the LEWIS and SENIOR checks"
- [methods] formula_denoising function definition: "The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion."
- [abstract] Spectral Denoising removes both noise types: "the Spectral Denoising algorithm removes both chemical and electronic noise"
- [discussion] Chemical plausibility without molecular connectivity: [evidence span withheld — anchor status: not_found]
- [results] [M+N2+H2O] reaction handling for aromatics: "Spectral Denoising accounts for possible [M+N2+H2O] molecule reactions when calculating the relative loss of fragment ions for substituted aromatic compounds"
