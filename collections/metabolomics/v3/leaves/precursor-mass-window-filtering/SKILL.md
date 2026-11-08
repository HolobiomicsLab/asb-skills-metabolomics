---
name: precursor-mass-window-filtering
description: 'Use when annotating unassigned query MS/MS spectra against a reference
  library: before Denoising Search scoring, filter library entries whose precursor
  m/z or molecular formula mass lies outside the instrument''s precursor mass accuracy
  window.'
when_to_use_negative:
- Input spectra lack reliable precursor m/z or accurate-mass data (e.g., low-resolution
  instruments), so no meaningful mass window can be defined
- Working on non-Orbitrap mass analyzers, where the paper notes parameters require
  retuning and revalidation rather than reusing the Orbitrap-optimized thresholds
- You only need library-wide spectral clustering/similarity and have no precursor-mass
  constraint to apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectral_denoising (sd.denoising_search / sd.denoising_search_batch)
  role: Applies the precursor mass accuracy window to restrict candidate formulas,
    then pseudo-denoises the query and scores only in-window candidates by entropy
    similarity
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: Formula_db
  role: Sorted candidate molecular formula database consulted for entries falling
    within the precursor mass accuracy window
- name: file_io.read_msp
  role: Reads query spectra and reference libraries, providing the 'precursor_mz'
    and 'smiles'/'formula' columns used for mass-window filtering
  repo: https://github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:5623c20945b6d617a075273b647bbf9d69ec5fa6aab22ad5b32117d7b73336ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - ClassyFire
  - Exploris 240 Orbitrap
  - Formula_db
  - Orbitrap Astral
  - file_io.read_msp
  - spectral_denoising (sd.denoising_search / sd.denoising_search_batch)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# precursor-mass-window-filtering

## Summary

Restricts candidate molecular formulas/compounds for spectral annotation to those whose precursor mass falls within a defined accuracy window (typically 5 ppm, or 10 mDa in Denoising Search), so that pseudo-denoising and entropy-similarity scoring are only computed against chemically plausible precursor candidates.

## When to use

Use when annotating unassigned query MS/MS spectra against a reference library: before Denoising Search scoring, filter library entries whose precursor m/z or molecular formula mass lies outside the instrument's precursor mass accuracy window.

## When NOT to use

- Input spectra lack reliable precursor m/z or accurate-mass data (e.g., low-resolution instruments), so no meaningful mass window can be defined
- Working on non-Orbitrap mass analyzers, where the paper notes parameters require retuning and revalidation rather than reusing the Orbitrap-optimized thresholds
- You only need library-wide spectral clustering/similarity and have no precursor-mass constraint to apply

## Inputs

- Query MS/MS spectra loaded from .msp files (peaks array plus precursor_mz column)
- Reference spectral library from .msp files with a 'smiles' (or formula) column
- Precursor mass accuracy threshold (10 mDa in the paper's Denoising Search; 5 ppm is the typical identity-search window)
- Sorted candidate formula database (Formula_db, sorted by mass)

## Outputs

- Per-spectrum table of in-window precursor candidates with columns 'query_peaks', 'query_peaks_denoised', 'entropy_similarity', and 'denoised_similarity'
- Aggregated annotation counts per spectrum/instrument after ranking the filtered candidates

## How to apply

Load query and reference MSP files with sd.read_msp, keeping the 'precursor_mz' column and a molecular-information column ('smiles' or formula fallback). For each query spectrum, restrict the reference library to candidates whose molecular formulas fall within the predefined precursor mass accuracy — the paper uses a 10-mDa threshold (a 5 ppm window is the typical default identity-search window) — before running sd.denoising_search or sd.denoising_search_batch. The restriction is what makes per-spectrum pseudo-denoising tractable: formula_denoising is only invoked for candidates inside the window, avoiding combinatorial explosion over the full formula space. Candidates outside the window are excluded entirely, mirroring how standard identity searches discard compounds failing accurate-mass matching. The function returns all in-window candidates, each with raw entropy_similarity and denoised_similarity, so ranking and annotation calls happen after, not during, the mass filtering step.

## Related tools

- **spectral_denoising (sd.denoising_search / sd.denoising_search_batch)** (Applies the precursor mass accuracy window to restrict candidate formulas, then pseudo-denoises the query and scores only in-window candidates by entropy similarity) — https://github.com/FanzhouKong/spectral_denoising
- **Formula_db** (Sorted candidate molecular formula database consulted for entries falling within the precursor mass accuracy window)
- **file_io.read_msp** (Reads query spectra and reference libraries, providing the 'precursor_mz' and 'smiles'/'formula' columns used for mass-window filtering) — https://github.com/FanzhouKong/spectral_denoising

## Examples

```
import spectral_denoising as sd; query_spectra = sd.read_msp('sample_data/query_spectra.msp'); reference_library = sd.read_msp('sample_data/reference_library.msp'); result = sd.denoising_search(query_spectra.iloc[0]['peaks'], query_spectra.iloc[0]['precursor_mz'], reference_library)
```

## Evaluation signals

- Every returned candidate's precursor m/z (or formula mass) lies within 10 mDa of the query precursor_mz; no candidate exceeds the window
- The result table contains the four required columns 'query_peaks', 'query_peaks_denoised', 'entropy_similarity', and 'denoised_similarity' with one row per candidate
- Reproducing the AD-plasma Orbitrap Astral vs Exploris 240 comparison yields approximately 2.3-fold (2.5-fold per the discussion) more annotated compounds on the Astral
- Denoised similarities for correct annotations outperform raw entropy similarities for the same candidate, indicating in-window pseudo-denoising worked
- Empty result sets only occur for queries with no plausible in-window formula, not from missing precursor_mz or molecular-information columns

## Limitations

- The window (10 mDa) cannot distinguish compounds whose exact masses differ by less than the instrument's mass accuracy; isomeric and isobaric candidates remain unresolved by this filter
- Reference spectra with inherently low spectral entropy are vulnerable to noise ions, potentially leading to false negatives even after correct window filtering
- In-source fragments and isomers share precursor masses and are not differentiated by mass-window filtering plus MS/MS spectral matching alone

## Evidence

- [results] 5 ppm typical window: "all identity-search algorithms already exclude compounds that do not match specific accurate mass windows of the precursor mass (typically at 5 ppm)"
- [results] 10-mDa threshold in Denoising Search: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [results] denoising over in-window formulas: "Denoising Search starts by denoising the experimental spectra using all molecular formulas that fall within the predefined precursor mass accuracy"
- [readme] candidates returned per query spectrum: [evidence span withheld — anchor status: external_doc]
