---
name: subformula-loss-annotation
description: Use when a peak list has already passed electronic denoising and a precursor
  formula (from SMILES+adduct or formula) is known, and remaining fragment ions must
  be classified as plausible subformula losses vs chemical noise.
when_to_use_negative:
- The precursor molecular formula cannot be retrieved from a reference spectral library
  or computed from SMILES — no master formula means no subformula enumeration
- The analyte may produce losses requiring molecular-connectivity knowledge (e.g.
  radical losses that Sirius/MS-FINDER also fail to recognize); plausibility checks
  here are limited to Seven Golden Rules without connectivity
- The input is a raw spectrum still containing electronic noise — run electronic_denoising
  first, since this skill assumes the fragment region is already cleaned
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Spectral Denoising
  role: Package providing formula_denoising, prep_formula, get_pmz_statistics, slice_spectrum,
    get_all_subformulas, check_candidates/get_denoise_tag and add_spectra
  repo: github.com/FanzhouKong/spectral_denoising
- name: seven_golden_rules
  role: check_ratio heuristic validating chemical plausibility of loss subformulas
    (rejects e.g. CH12; LEWIS/SENIOR checks ignored)
  repo: github.com/FanzhouKong/spectral_denoising
- name: chemparse
  role: Parsing subformula compositions during candidate enumeration and mass calculation
- name: molmass
  role: Computing exact masses of candidate subformula losses
- name: rdkit
  role: Detecting benzene substructures to add N2/O to the master formula for [M+N2+H2O]
    aromatic reactions
- name: numpy
  role: Storing and manipulating the [mz, intensity] peak arrays
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:bd32bb92d28b3e43c6a229c5360d087f17b6e4a489099d1bfc59878b71d4adcd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spectral Denoising
  - chemparse
  - molmass
  - numpy
  - rdkit
  - seven_golden_rules
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# subformula-loss-annotation

## Summary

Tags each fragment ion in an electronically denoised MS/MS spectrum as chemical signal or chemical noise by testing whether its exact mass corresponds to a chemically plausible subformula loss from the precursor ion, validated with the Seven Golden Rules.

## When to use

Use when a peak list has already passed electronic denoising and a precursor formula (from SMILES+adduct or formula) is known, and remaining fragment ions must be classified as plausible subformula losses vs chemical noise.

## When NOT to use

- The precursor molecular formula cannot be retrieved from a reference spectral library or computed from SMILES — no master formula means no subformula enumeration
- The analyte may produce losses requiring molecular-connectivity knowledge (e.g. radical losses that Sirius/MS-FINDER also fail to recognize); plausibility checks here are limited to Seven Golden Rules without connectivity
- The input is a raw spectrum still containing electronic noise — run electronic_denoising first, since this skill assumes the fragment region is already cleaned

## Inputs

- Electronically denoised peak array (numpy float32 [mz, intensity]) from electronic_denoising
- Precursor molecular structure as SMILES string (or molecular formula string)
- Adduct string, e.g. '[M+Na]+' or '[M+H]+'
- Precursor m/z
- Formula database sorted by mass (Formula_db) for Denoising Search use cases

## Outputs

- Chemically denoised peak array (formula_denoised_spectrum.npy) containing only ions with plausible subformula losses plus the re-attached precursor region
- Per-ion denoise tags ('True'/'False') from get_denoise_tag

## How to apply

Derive the master precursor formula with prep_formula from the SMILES and adduct (e.g. 'O=c1nc[nH]c2nc[nH]c12' with '[M+Na]+' gives 'C5H4N4NaO'; add N2/O for benzene substructures to account for [M+N2+H2O] reactions in substituted aromatics). Estimate precursor statistics with get_pmz_statistics using an initial mass_tolerance of 0.005 Da (auto-inflated up to a +/- 10 mDa precursor search window), then slice the spectrum with slice_spectrum so only the fragment region is denoised. Enumerate candidate subformulas and exact masses from the master formula with get_all_subformulas, sorted by mass to accelerate search. For each fragment ion, call check_candidates/get_denoise_tag, validating loss subformulas with the seven_golden_rules.check_ratio heuristic — rejecting impossible compositions like CH12 — while deliberately ignoring the LEWIS and SENIOR checks designed for intact molecules. Retain only ions tagged 'True', re-attach the intact precursor region with add_spectra, and save the array (e.g. formula_denoised_spectrum.npy).

## Related tools

- **Spectral Denoising** (Package providing formula_denoising, prep_formula, get_pmz_statistics, slice_spectrum, get_all_subformulas, check_candidates/get_denoise_tag and add_spectra) — github.com/FanzhouKong/spectral_denoising
- **seven_golden_rules** (check_ratio heuristic validating chemical plausibility of loss subformulas (rejects e.g. CH12; LEWIS/SENIOR checks ignored)) — github.com/FanzhouKong/spectral_denoising
- **chemparse** (Parsing subformula compositions during candidate enumeration and mass calculation)
- **molmass** (Computing exact masses of candidate subformula losses)
- **rdkit** (Detecting benzene substructures to add N2/O to the master formula for [M+N2+H2O] aromatic reactions)
- **numpy** (Storing and manipulating the [mz, intensity] peak arrays)

## Examples

```
peak_denoised = sd.spectral_denoising(peak_with_noise, 'O=c1nc[nH]c2nc[nH]c12', '[M+Na]+'); np.save('formula_denoised_spectrum.npy', peak_denoised)
```

## Evaluation signals

- Entropy similarity of the chemically denoised spectrum against the clean reference reaches ~1.00 (demo reports 0.33 with chemical noise vs 1.00 after denoising)
- Every retained fragment ion has a loss subformula passing seven_golden_rules.check_ratio; impossible compositions like CH12 are absent from the retained set
- The intact precursor region is preserved verbatim — only the fragment region is modified (verified after add_spectra re-attachment)
- Average similarity of denoised NIST23 spectra exceeds 0.99 in the paper's benchmark on 240 tested metabolites
- No fragment exceeds the precursor mass window defined by get_pmz_statistics (initial 0.005 Da tolerance, max +/- 10 mDa)

## Limitations

- Chemical plausibility checks are limited to the Seven Golden Rules method without considering molecular connectivity, so some chemically real but connectivity-dependent losses are incorrectly removed
- Not all artificially added noise ions were removed in benchmarking for the same connectivity reason
- LEWIS and SENIOR checks are intentionally ignored because they are designed for intact molecules, not fragment losses
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Parameters are optimized for Orbitrap instruments; adapting to other mass analyzers requires retuning and validation

## Evidence

- [results] Seven Golden Rules validation of loss subformulas: "The chemical plausibility of relative loss subformulas was validated using the Seven Golden Rules algorithm to discard chemically impossible losses"
- [methods] formula_denoising definition: "The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion."
- [results] Chemical denoising rationale: "chemical denoising identifies and removes chemical noise ions by calculating whether the exact mass of each fragment can logically be associated with a subformula loss from the parent molecular ion"
- [results] Aromatic [M+N2+H2O] handling: "Spectral Denoising accounts for possible [M+N2+H2O] molecule reactions when calculating the relative loss of fragment ions for substituted aromatic compounds"
- [discussion] Connectivity limitation: "our chemical plausibility checks were limited to the algorithms embedded in the Seven Golden Rules method32, without considering molecular connectivity"
- [readme] Package installation and Python version: "Please use Python version between 3.8 to 3.12 for this package to work. RDkit currently does not have a distribution compitable to python 3.13!!!!!"
