---
name: subformula-loss-chemical-plausibility-filtering
description: Use when cleaning MS/MS spectra of chemical noise ions with a known or
  predicted precursor molecular formula (SMILES or formula) and adduct, so fragments
  unexplainable by plausible subformula losses can be discarded before spectral matching.
when_to_use_negative:
- No precursor molecular formula or SMILES is available and none can be retrieved
  from a reference library or cheminformatic prediction, since the filter cannot evaluate
  subformula losses
- The data come from non-Orbitrap mass analyzers, where the method's parameters must
  be retuned and revalidated before use
- Reference spectra have inherently low spectral entropy (<0.5, fewer than two fragment
  ions), making them vulnerable to noise-ion-driven false negatives even after filtering
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: spectral-denoising (formula_denoising)
  role: Implements chemical noise removal by checking whether each fragment ion mass
    can be formed from a chemically plausible subformula loss to the precursor ion;
    accepts SMILES or formula input
  repo: github.com/FanzhouKong/spectral_denoising
- name: Seven Golden Rules
  role: Validates the chemical plausibility of relative loss subformulas, discarding
    chemically impossible losses (e.g. CH12) while ignoring LEWIS and SENIOR checks
- name: spectral_denoising.spectral_operations
  role: Provides spectral_entropy/normalized_entropy and entropy_similarity helpers
    and remove_precursor preprocessing used to score the chemically filtered spectra
  repo: github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:36f5a388e64f0aa8e05e80a3637a8388de3087ba7b1c10e733eb0aef37976949
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - MS Reduce
  - Seven Golden Rules
  - spectral-denoising (formula_denoising)
  - spectral_denoising.spectral_operations
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Subformula-loss chemical plausibility filtering

## Summary

A chemical noise removal step in MS/MS spectral denoising that keeps only fragment ions whose exact masses can be explained as chemically plausible subformula losses from the parent molecular ion, validated with the Seven Golden Rules algorithm.

## When to use

Use when cleaning MS/MS spectra of chemical noise ions with a known or predicted precursor molecular formula (SMILES or formula) and adduct, so fragments unexplainable by plausible subformula losses can be discarded before spectral matching.

## When NOT to use

- No precursor molecular formula or SMILES is available and none can be retrieved from a reference library or cheminformatic prediction, since the filter cannot evaluate subformula losses
- The data come from non-Orbitrap mass analyzers, where the method's parameters must be retuned and revalidated before use
- Reference spectra have inherently low spectral entropy (<0.5, fewer than two fragment ions), making them vulnerable to noise-ion-driven false negatives even after filtering

## Inputs

- MS/MS peak arrays (mz/intensity) read via sd.read_msp from .msp files
- Precursor molecular formula or SMILES string per compound
- Adduct annotation (e.g. [M+Na]+) for precursor m/z calculation
- Precursor m/z
- Formula_db (mass-sorted candidate formula database) for Denoising Search variants

## Outputs

- Denoised peak arrays with chemical noise ions removed (list from spectral_denoising_batch)
- Entropy similarity scores of denoised spectra against ground-truth or reference spectra
- Per-compound similarity aggregates across dilution stages (e.g. benchmark_results.csv)

## How to apply

For each query spectrum, take the precursor molecular formula (retrieved from the SMILES/adduct or from reference library top hits) and test whether each fragment's exact mass corresponds to a chemically plausible subformula loss from the parent ion. Validate candidate loss subformulas with the Seven Golden Rules algorithm to discard chemically impossible losses (e.g. CH12), deliberately ignoring the LEWIS and SENIOR checks; for substituted aromatic compounds, also account for possible [M+N2+H2O] molecule reactions when computing relative losses. In the spectral_denoising package this is implemented as the formula_denoising function, which accepts either SMILES or formula input and is applied after electronic_denoising; the precursor mass tolerance starts at 5 ppm with a maximum of 10 mDa. Run it in batch mode via spectral_denoising_batch (wrapped in main() because it parallelizes with multiprocessing), then score the denoised spectra against ground-truth or library spectra using entropy similarity, ignoring residual precursor ion intensity.

## Related tools

- **spectral-denoising (formula_denoising)** (Implements chemical noise removal by checking whether each fragment ion mass can be formed from a chemically plausible subformula loss to the precursor ion; accepts SMILES or formula input) — github.com/FanzhouKong/spectral_denoising
- **Seven Golden Rules** (Validates the chemical plausibility of relative loss subformulas, discarding chemically impossible losses (e.g. CH12) while ignoring LEWIS and SENIOR checks)
- **spectral_denoising.spectral_operations** (Provides spectral_entropy/normalized_entropy and entropy_similarity helpers and remove_precursor preprocessing used to score the chemically filtered spectra) — github.com/FanzhouKong/spectral_denoising

## Examples

```
peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct); print(f'{entropy_similairty(peak_denoised, peak, pmz=pmz):.2f}')
```

## Evaluation signals

- Denoised spectra should show reduced spectral entropy versus the noisy input (paper example: S drops from 3.3 to 1.9) and entropy similarity >0.99 on average for validated NIST23 spectra
- In the 240-metabolite dilution benchmark, chemically filtered spectra should reach entropy similarity >0.75 at ~35-fold lower injected quantities than MS Reduce, 1% base-peak thresholding and DNL, with zero chemical-noise-induced false negatives (MS Reduce and DNL caused 107 and 84 false negatives)
- The filter must be robust when noise ion intensity exceeds true fragment ion intensity by >150-fold
- No fragment ion explaining a true subformula loss should be removed — check that denoised spectra retain the true fragment ions of the ground-truth 500-pmol spectra
- Precursor mass tolerance check: candidate formulas must fall within the 5 ppm (max 10 mDa) precursor accuracy window

## Limitations

- Chemical plausibility checks are limited to the Seven Golden Rules algorithm and do not consider molecular connectivity, so not all added noise ions are removed
- Not all artificially added noise ions are removed for this same reason; reference spectra with low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Optimized for Orbitrap instruments; adapting to other mass analyzers requires retuning and validating parameters
- Requires Python 3.8–3.12 because RDKit has no compatible distribution for Python 3.13

## Evidence

- [results] chemical denoising identifies and removes chemical noise ions by calculating whether the exact mass of each fragment can logically be associated with a subformula loss from the parent molecular ion: "chemical denoising identifies and removes chemical noise ions by calculating whether the exact mass of each fragment can logically be associated with a subformula loss from the parent molecular ion"
- [results] Seven Golden Rules validation: "validated using the Seven Golden Rules algorithm to discard chemically impossible losses (for example, CH12)32 while ignoring the LEWIS and SENIOR checks"
- [results] aromatic N2/H2O reaction handling: "Spectral Denoising accounts for possible [M+N2+H2O] molecule reactions when calculating the relative loss of fragment ions for substituted aromatic compounds"
- [methods] formula_denoising function description: "The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion."
- [discussion] connectivity limitation: "our chemical plausibility checks were limited to the algorithms embedded in the Seven Golden Rules method32, without considering molecular connectivity"
- [readme] batch-mode usage from README: [evidence span withheld — anchor status: external_doc]
