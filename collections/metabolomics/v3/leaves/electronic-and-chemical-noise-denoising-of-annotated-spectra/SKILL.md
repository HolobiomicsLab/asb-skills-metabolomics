---
name: electronic-and-chemical-noise-denoising-of-annotated-spectra
description: Use when query MS/MS spectra (e.g. MSP files) contain electronic or chemical
  noise that lowers entropy similarity against reference library spectra, and the
  candidate compound's SMILES or molecular formula and adduct are known.
when_to_use_negative:
- The molecular formula or SMILES of the candidate compound is unknown — classic denoising
  requires known precursor structure; use Denoising Search instead, which denoises
  against all formulas within the precursor mass window
- Data were acquired on non-Orbitrap instruments — the article states the method is
  optimized for Orbitrap instruments and adaptation requires retuning and validating
  parameters
- Reference spectra have inherently low spectral entropy (few fragment ions) — these
  are particularly vulnerable to noise ions and can yield false negatives even after
  denoising
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: Core algorithm combining electronic_denoising and formula_denoising to remove
    electronic and chemical noise ions from annotated MS/MS spectra
  repo: github.com/FanzhouKong/spectral_denoising
- name: electronic_denoising
  role: Removes electronic ('grass') noise ions with identical intensities greater
    than 4, independent of relative intensity
  repo: github.com/FanzhouKong/spectral_denoising
- name: formula_denoising
  role: Removes chemical noise ions by evaluating chemically plausible subformula
    losses to the precursor ion
  repo: github.com/FanzhouKong/spectral_denoising
- name: MS Reduce
  role: Comparison denoising method used as a baseline; introduced 107 false negatives
    by removing true fragment ions
- name: DNL denoising
  role: Comparison dynamic noise level denoising baseline; introduced 84 false negatives
    by removing true fragment ions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:cbfc6ef824f40bf5958b66f167946ed6533acf0e5a92543b9ae5e99f539b2499
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - DNL denoising
  - MS Reduce
  - Spectral Denoising
  - electronic_denoising
  - formula_denoising
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# electronic-and-chemical-noise-denoising-of-annotated-spectra

## Summary

Applies the Spectral Denoising algorithm (electronic_denoising followed by formula_denoising) to remove both electronic 'grass' noise ions and chemically implausible fragment ions from MS/MS spectra of compounds with known SMILES/formula and adduct, improving entropy-similarity spectral matching.

## When to use

Use when query MS/MS spectra (e.g. MSP files) contain electronic or chemical noise that lowers entropy similarity against reference library spectra, and the candidate compound's SMILES or molecular formula and adduct are known.

## When NOT to use

- The molecular formula or SMILES of the candidate compound is unknown — classic denoising requires known precursor structure; use Denoising Search instead, which denoises against all formulas within the precursor mass window
- Data were acquired on non-Orbitrap instruments — the article states the method is optimized for Orbitrap instruments and adaptation requires retuning and validating parameters
- Reference spectra have inherently low spectral entropy (few fragment ions) — these are particularly vulnerable to noise ions and can yield false negatives even after denoising

## Inputs

- MS/MS query spectra in .msp format (peaks, precursor_mz, smiles or formula, adduct columns) read via sd.read_msp
- Reference/ground-truth spectra (e.g. 500-pmol standard spectra or NIST23/MassBank.us/GNPS library entries) in .msp format
- Compound structural identifiers: SMILES string or molecular formula, plus adduct (e.g. [M+Na]+)
- Formula_db for chemical plausibility checks (sorted by mass)

## Outputs

- Denoised peak arrays (mass, intensity) with electronic and chemical noise ions removed
- Entropy similarity scores of denoised vs reference spectra (e.g. similarity >0.75 threshold)
- Batch denoising results returned as a list of denoised spectra
- Benchmark summaries such as explained ion intensity and lowest injected quantity reaching entropy similarity >0.75 (benchmark_results.csv)

## How to apply

Load query and reference spectra with sd.read_msp, then sanitize each spectrum (remove zero-intensity ions, sort by mass) and exclude compounds with fewer than two fragment ions (spectral entropy <0.5). First run electronic_denoising, which removes electronic noise ions using the NIST23-derived threshold that more than four ions with identical intensities within ±0.1% relative intensities are extremely unlikely. Then run formula_denoising with the known SMILES (or formula) and adduct, testing whether each fragment's exact mass corresponds to a chemically plausible subformula loss from the precursor, validating losses with the Seven Golden Rules algorithm while ignoring LEWIS and SENIOR checks, and accounting for [M+N2+H2O] reactions for substituted aromatic compounds. The denoised spectrum can then be scored against ground-truth spectra via entropy_similarity, ignoring remaining precursor ion intensity.

## Related tools

- **Spectral Denoising** (Core algorithm combining electronic_denoising and formula_denoising to remove electronic and chemical noise ions from annotated MS/MS spectra) — github.com/FanzhouKong/spectral_denoising
- **electronic_denoising** (Removes electronic ('grass') noise ions with identical intensities greater than 4, independent of relative intensity) — github.com/FanzhouKong/spectral_denoising
- **formula_denoising** (Removes chemical noise ions by evaluating chemically plausible subformula losses to the precursor ion) — github.com/FanzhouKong/spectral_denoising
- **MS Reduce** (Comparison denoising method used as a baseline; introduced 107 false negatives by removing true fragment ions)
- **DNL denoising** (Comparison dynamic noise level denoising baseline; introduced 84 false negatives by removing true fragment ions)

## Examples

```
peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct); sim = entropy_similairty(peak_denoised, peak, pmz = pmz)
```

## Evaluation signals

- Denoised NIST23 spectra should reach average entropy similarity >0.99 against their reference spectra
- Spectral entropy of contaminated spectra (e.g. S = 3.3) should decrease after denoising (e.g. S = 1.9)
- On the 240-metabolite dilution benchmark, >700 previously unmatchable spectra should lift to entropy similarity >0.75 and annotation rates should increase by ~20%
- No true fragment ions should be lost: unlike MS Reduce (107 false negatives) and DNL (84 false negatives), Spectral Denoising should introduce essentially no false negatives
- Robustness check: denoising should remain correct even with >150-fold higher intensity of noise ions than true fragment ions

## Limitations

- Chemical plausibility checks are limited to Seven Golden Rules without considering molecular connectivity, so not all artificially added noise ions are removed
- Method is optimized for Orbitrap instruments; adapting to other mass analyzers requires retuning and validating parameters
- Requires known SMILES or molecular formula and adduct for the query compound; single-peak low-entropy reference spectra remain vulnerable to residual noise ions
- Requires Python >=3.8 and <3.13 because RDKit has no compatible distribution for Python 3.13

## Evidence

- [results] electronic denoising removes electronic noise ions using NIST23 threshold: "These data served as a valid threshold to automatically identify electronic noise ions in experimental MS/MS spectra and remove these ions, independent of the relative intensity"
- [results] chemical denoising via subformula losses: "chemical denoising identifies and removes chemical noise ions by calculating whether the exact mass of each fragment can logically be associated with a subformula loss from the parent molecular ion"
- [results] Seven Golden Rules validation with LEWIS/SENIOR ignored: "validated using the Seven Golden Rules algorithm to discard chemically impossible losses (for example, CH12)32 while ignoring the LEWIS and SENIOR checks"
- [methods] formula_denoising function description: "The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion."
- [results] exclude low-fragmentation compounds: "A total of 28 compounds were excluded due to insufficient fragmentation with fewer than two fragment ions (spectral entropy <0.5)"
- [other] benchmark result vs alternatives: [evidence span withheld — anchor status: not_found]
- [readme] Python version constraint: "Please use Python version between 3.8 to 3.12 for this package to work. RDkit currently does not have a distribution compitable to python 3.13"
