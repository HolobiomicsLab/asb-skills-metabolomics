---
name: spectral-library-search-annotation
description: Use when an experimental MS/MS spectrum (query_spectra.msp with 'peaks'
  and 'precursor_mz') must be annotated against a reference library (reference_library.msp)
  and the true precursor molecular formula is unknown, so all formula candidates within
  mass accuracy must be scored.
when_to_use_negative:
- The precursor molecular formula is already known with certainty — plain spectral_denoising
  on the single formula is the appropriate path instead of denoising_search over all
  candidates
- Input is already an annotated feature/compound table (no raw MS/MS 'peaks' arrays
  to match) — library search adds nothing
- Data are from non-Orbitrap mass analyzers — the article notes the method is optimized
  for Orbitrap instruments and adapting it requires retuning and validating parameters
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Denoising Search
  role: 'Core algorithm: pseudo-denoises the query spectrum using every molecular
    formula within the precursor mass accuracy and scores all candidates by entropy
    similarity'
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: Spectral Denoising
  role: Provides the underlying electronic_denoising and formula_denoising functions
    that remove electronic and chemical noise ions for each candidate formula
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: spectral_denoising (Python package)
  role: Implements read_msp, denoising_search, denoising_search_batch, spectral_operations.remove_precursor/remove_zero_ions,
    and entropy similarity scoring
  repo: https://github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:1cad4f45b5633783c03b331630b1856a6585b55f9b0ea77e273bc1df44c37fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - Denoising Search
  - Spectral Denoising
  - spectral_denoising (Python package)
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# spectral-library-search-annotation

## Summary

Annotate an experimental MS/MS spectrum by searching it against a reference spectral library without assuming a single precursor molecular formula: pseudo-denoise the query against every candidate formula within the precursor mass window, then score matches with entropy similarity computed on both raw and denoised spectra.

## When to use

Use when an experimental MS/MS spectrum (query_spectra.msp with 'peaks' and 'precursor_mz') must be annotated against a reference library (reference_library.msp) and the true precursor molecular formula is unknown, so all formula candidates within mass accuracy must be scored.

## When NOT to use

- The precursor molecular formula is already known with certainty — plain spectral_denoising on the single formula is the appropriate path instead of denoising_search over all candidates
- Input is already an annotated feature/compound table (no raw MS/MS 'peaks' arrays to match) — library search adds nothing
- Data are from non-Orbitrap mass analyzers — the article notes the method is optimized for Orbitrap instruments and adapting it requires retuning and validating parameters

## Inputs

- query_spectra.msp (MSP file with 'peaks' and 'precursor_mz' columns)
- reference_library.msp (MSP file with 'peaks', 'smiles'/'formula', 'adduct', 'precursor_mz' columns)
- Precursor mass accuracy threshold (10 mDa as used in the article)

## Outputs

- Per-candidate result table with columns 'query_peaks', 'query_peaks_denoised', 'entrpy_similarity', 'denoised_similarity'
- denoising_search_results.csv (all precursor candidates of the query spectrum, each with raw and denoised entropy similarities)
- Batch mode: list of per-spectrum candidate tables from denoising_search_batch

## How to apply

Load query_spectra.msp and reference_library.msp with spectral_denoising.read_msp, extracting the 'peaks' and 'precursor_mz' columns. Restrict the reference library to candidates whose molecular formulas fall within the precursor mass accuracy (10-mDa threshold per the article). For each candidate, pseudo-denoise the query spectrum against that candidate's molecular information using formula_denoising (subformula-loss check with Seven Golden Rules validation, ignoring LEWIS and SENIOR checks). Compute entropy similarity between query and reference spectra for both the raw and denoised query, after removing zero-intensity ions and excluding the precursor ion region (spectral_operations.remove_precursor) so scoring focuses entirely on genuine fragment spectra. Return all candidates as a table with 'query_peaks', 'query_peaks_denoised', 'entrpy_similarity', and 'denoised_similarity' columns, rather than a single best hit — downstream decisions (retention time matching, cross-section comparison, metadata screening) should be layered on top.

## Related tools

- **Denoising Search** (Core algorithm: pseudo-denoises the query spectrum using every molecular formula within the precursor mass accuracy and scores all candidates by entropy similarity) — https://github.com/FanzhouKong/spectral_denoising
- **Spectral Denoising** (Provides the underlying electronic_denoising and formula_denoising functions that remove electronic and chemical noise ions for each candidate formula) — https://github.com/FanzhouKong/spectral_denoising
- **spectral_denoising (Python package)** (Implements read_msp, denoising_search, denoising_search_batch, spectral_operations.remove_precursor/remove_zero_ions, and entropy similarity scoring) — https://github.com/FanzhouKong/spectral_denoising

## Examples

```
import spectral_denoising as sd; query_spectra = sd.read_msp('sample_data/query_spectra.msp'); reference_library = sd.read_msp('sample_data/reference_library.msp'); result = sd.denoising_search(query_spectra.iloc[0]['peaks'], query_spectra.iloc[0]['precursor_mz'], reference_library)
```

## Evaluation signals

- denoising_search_results.csv exists and contains all four required columns ('query_peaks', 'query_peaks_denoised', 'entrpy_similarity', 'denoised_similarity') with one row per candidate in the mass window
- Every returned candidate's precursor falls within the 10-mDa mass accuracy window of the query precursor m/z
- Denoised spectral entropy of the query drops relative to the raw spectrum (article example: S = 1.9 denoised vs S = 3.3 contaminated), and similarity of denoised NIST23 spectra exceeded 0.99 on average
- Precursor ion region is absent from similarity calculations (remove_precursor applied), so scores reflect only genuine fragment ions
- Single-spectrum denoising_search returns multiple candidate rows rather than a single hit, since no single formula is assumed

## Limitations

- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Inability to differentiate isomeric compounds and in-source fragments when relying solely on MS/MS spectral matching; hard thresholding on spectral similarity alone is suboptimal — supplement with retention time matching, molecular cross-section comparisons, and biological metadata screening
- Chemical plausibility checks are limited to the Seven Golden Rules without considering molecular connectivity, so not all noise ions are removed; method is optimized for Orbitrap instruments and requires retuning for other mass analyzers
- Requires Python >= 3.8 and < 3.13 (RDkit has no Python 3.13-compatible distribution)

## Evidence

- [results] Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy: "Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy"
- [results] 10-mDa threshold: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [methods] remove_precursor: "``spectral_operations.remove_precursor`` will remove the precursor ion region from the query spectrum."
- [readme] README on denoising_search return value: [evidence span withheld — anchor status: external_doc]
