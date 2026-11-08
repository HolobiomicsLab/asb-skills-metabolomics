---
name: msms-noise-denoising
description: Use when raw MS/MS query spectra (e.g., AD plasma from Orbitrap Astral
  or Exploris 240) contain electronic and chemical noise ions, and the goal is higher-confidence
  compound annotation via spectral library matching with entropy similarity.
when_to_use_negative:
- Mass analyzer is not an Orbitrap instrument — the method is optimized for Orbitrap
  and adapting it requires retuning and validating parameters
- The analysis goal requires differentiating isomeric compounds or in-source fragments,
  or compound confirmation via retention time / cross-section / biological metadata
  — MS/MS-only matching after denoising cannot resolve these
- Reference spectra or query spectra have inherently low spectral entropy (e.g., <0.5,
  fewer than two fragment ions) — such spectra are vulnerable to noise ions and prone
  to false negatives
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: spectral_denoising (sd)
  role: Core Python package providing sd.read_msp, sd.spectral_denoising, sd.spectral_denoising_batch,
    sd.denoising_search, sd.denoising_search_batch, and spectral_operations (remove_zero_ions,
    remove_precursor)
  repo: github.com/FanzhouKong/spectral_denoising
- name: Denoising Search
  role: Pseudo-denoises query spectra using all molecular formulas within the 10-mDa
    precursor mass accuracy, then scores raw vs denoised entropy similarity against
    reference library candidates
  repo: github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Entropy similarity metric used to compare raw and denoised query spectra against
    reference spectra
- name: ClassyFire
  role: Compound superclassification of recovered annotations (e.g., drug metabolites,
    household/industrial chemicals, pesticides)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:9c0c5e8756c9704919e44689bf6b47b56771a0b9c353774e76f2ab0fe6efdc05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - Denoising Search
  - Exploris 240 Orbitrap
  - Orbitrap Astral
  - Spectral Denoising
  - ms_entropy
  - sd.denoising_search
  - sd.denoising_search_batch
  - spectral_denoising (sd)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5238095238095238
n_claims: 21
---

# msms-noise-denoising

## Summary

Two-stage MS/MS spectrum cleaning — electronic denoising (removing 'grass' noise ions) and chemical denoising (removing ions not explainable by plausible subformula losses) — plus Denoising Search, which pseudo-denoises query spectra against reference library candidates to boost compound annotations in plasma metabolomics/exposomics on Orbitrap instruments.

## When to use

Use when raw MS/MS query spectra (e.g., AD plasma from Orbitrap Astral or Exploris 240) contain electronic and chemical noise ions, and the goal is higher-confidence compound annotation via spectral library matching with entropy similarity.

## When NOT to use

- Mass analyzer is not an Orbitrap instrument — the method is optimized for Orbitrap and adapting it requires retuning and validating parameters
- The analysis goal requires differentiating isomeric compounds or in-source fragments, or compound confirmation via retention time / cross-section / biological metadata — MS/MS-only matching after denoising cannot resolve these
- Reference spectra or query spectra have inherently low spectral entropy (e.g., <0.5, fewer than two fragment ions) — such spectra are vulnerable to noise ions and prone to false negatives

## Inputs

- Query MS/MS spectra in MSP format (.msp) with 'peaks' and 'precursor_mz' columns
- Reference spectral library in MSP format (e.g., NIST23, MassBank.us, GNPS subsets) exposing a 'smiles' column or a molecular formula column
- Known compound identity (SMILES or molecular formula) and adduct (e.g., '[M+Na]+') for classic spectral denoising on a single spectrum
- formula_db sorted by precursor mass for formula-restricted Denoising Search

## Outputs

- Denoised peak arrays (query_peaks_denoised) with reduced spectral entropy
- Per-spectrum candidate annotations with entropy similarity of the raw query and denoised_similarity of the denoised query
- Aggregated annotation counts and compound-class breakdowns (e.g., ClassyFire superclasses: drug metabolites, household/industrial chemicals, pesticides)
- Full per-spectrum candidate/denoised-similarity table (e.g., denoising_search_results.csv)

## How to apply

Load query spectra and reference library from MSP files with sd.read_msp, which sorts peaks and removes zero-intensity ions. Apply electronic denoising to remove 'grass' noise using the NIST23-derived threshold (fewer than four ions within ±0.1% relative intensities), then chemical denoising via sd.spectral_denoising(query_peaks, smiles/formula, adduct): check whether each fragment's exact mass maps to a chemically plausible subformula loss from the precursor, validated with the Seven Golden Rules algorithm (ignoring LEWIS and SENIOR checks), and account for [M+N2+H2O] reactions for substituted aromatic compounds. For annotation without a known formula, run sd.denoising_search / sd.denoising_search_batch, which denoises the query using every molecular formula within the 10-mDa precursor mass accuracy and scores both raw and denoised spectra by entropy similarity against library candidates, ignoring remaining precursor ion intensity. Batch functions use multiprocessing, so wrap calls in a main() function when running from scripts.

## Related tools

- **spectral_denoising (sd)** (Core Python package providing sd.read_msp, sd.spectral_denoising, sd.spectral_denoising_batch, sd.denoising_search, sd.denoising_search_batch, and spectral_operations (remove_zero_ions, remove_precursor)) — github.com/FanzhouKong/spectral_denoising
- **Denoising Search** (Pseudo-denoises query spectra using all molecular formulas within the 10-mDa precursor mass accuracy, then scores raw vs denoised entropy similarity against reference library candidates) — github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Entropy similarity metric used to compare raw and denoised query spectra against reference spectra)
- **ClassyFire** (Compound superclassification of recovered annotations (e.g., drug metabolites, household/industrial chemicals, pesticides))

## Examples

```
import spectral_denoising as sd; query_spectra = sd.read_msp('sample_data/query_spectra.msp'); reference_library = sd.read_msp('sample_data/reference_library.msp'); result = sd.denoising_search(query_spectra.iloc[0]['peaks'], query_spectra.iloc[0]['precursor_mz'], reference_library)
```

## Evaluation signals

- Benchmark on ~240 tested metabolites: Spectral Denoising should outperform 1% base-peak thresholding, DNL denoising, and MS Reduce; denoised NIST23 spectra should reach average similarity >0.99
- Spectral entropy of the denoised spectrum should drop measurably (paper example: S = 1.9 denoised vs S = 3.3 contaminated)
- Robustness check: denoising should succeed even with >150-fold higher intensity of noise ions than true fragment ions
- On AD plasma MS/MS, Denoising Search should yield roughly 2.3–2.5-fold more annotated compounds on Orbitrap Astral vs Exploris 240, recovering the expected exposome compound classes
- Precondition check: spectra with spectral entropy <0.5 (fewer than two fragment ions) should be excluded, as done for 28 compounds in the dilution-series analysis

## Limitations

- Chemical plausibility checks are limited to the Seven Golden Rules algorithm without molecular connectivity, so some added noise ions may not be removed (e.g., impossible losses like CH12 are caught, but connectivity-blind checks leave gaps)
- Method is optimized for Orbitrap instruments; other mass analyzers require retuning and validating parameters
- Reference spectra with inherently low spectral entropy are vulnerable to noise ions, potentially leading to false negatives; MS/MS-only matching cannot differentiate isomers or in-source fragments even after denoising
- Python version must be between 3.8 and 3.12 — RDKit has no distribution compatible with Python 3.13

## Evidence

- [results] Electronic denoising threshold: "For more than 99.5% of all ions per spectrum, fewer than four ions were found within relative intensities of ±0.1%"
- [results] Chemical denoising rationale: "chemical denoising identifies and removes chemical noise ions by calculating whether the exact mass of each fragment can logically be associated with a subformula loss from the parent molecular ion"
- [results] Seven Golden Rules validation: "The chemical plausibility of relative loss subformulas was validated using the Seven Golden Rules algorithm to discard chemically impossible losses"
- [results] Denoising Search mechanism: "Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy"
- [readme] Batch search API in README: [evidence span withheld — anchor status: external_doc]
- [readme] Python version constraint in README: "Please use Python version between 3.8 to 3.12 for this package to work. RDkit currently does not have a distribution compitable to python 3.13!!!!!"
- [methods] Multiplicity threshold justification: "According to empiracally tested on NIST23 database, in a given spectrum, the number of ions with identical intensities more than 4 is extremely unlikely"
