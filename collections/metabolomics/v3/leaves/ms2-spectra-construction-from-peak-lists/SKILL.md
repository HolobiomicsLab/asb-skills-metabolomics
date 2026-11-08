---
name: ms2-spectra-construction-from-peak-lists
description: Use when you have raw MS2 peak lists (paired m/z and intensity vectors)
  with known precursor m/z values and need to assemble them into Spectra objects for
  comparison or conversion, rather than reading from mzML/MGF files.
when_to_use_negative:
- Input is already an mzML or MGF file — use Spectra's file-based backends to import
  spectra instead of hand-building peak lists.
- Data are MS1 scans or profile-mode spectra; this skill targets centroided MS2 peak
  lists with explicit precursorMz metadata.
- The goal is conversion from Python matchms.Spectrum objects back to R — use SpectriPy's
  dedicated conversion functions rather than manual construction.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Spectra
  role: Defines the Spectra::Spectra container and MsBackendDataFrame used to hold
    the peak-list-derived MS2 spectra; provides compareSpectra() for the built-in
    scoring step that consumes these objects
  repo: https://github.com/RforMassSpectrometry/Spectra
- name: SpectriPy
  role: Converts the constructed Spectra objects to matchms.Spectrum objects so matchms
    scoring functions (e.g., CosineHungarian, CosineGreedy) can be applied from R
  repo: https://github.com/rformassspectrometry/SpectriPy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/joss.08070
    title: 'SpectriPy: Enhancing Cross-Language Mass Spectrometry Data Analysis with
      R and Python'
  source_content_hash: sha256:b731b0a3e395e003fea57bd2123a29c66f66915ffee49013ea36ebfe92c0b51c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RforMassSpectrometry/Spectra
  tools:
  - R
  - Spectra
  - SpectriPy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# MS2 spectra construction from peak lists

## Summary

Builds R Spectra::Spectra objects for MS2 spectra directly from m/z and intensity peak lists embedded in a DataFrame (e.g., the Caffeine precursorMz 195.0877 and 1-Methylhistidine precursorMz 170.0924, HMDB0000001 vignette examples), providing the input objects required for downstream similarity comparisons.

## When to use

Use when you have raw MS2 peak lists (paired m/z and intensity vectors) with known precursor m/z values and need to assemble them into Spectra objects for comparison or conversion, rather than reading from mzML/MGF files.

## When NOT to use

- Input is already an mzML or MGF file — use Spectra's file-based backends to import spectra instead of hand-building peak lists.
- Data are MS1 scans or profile-mode spectra; this skill targets centroided MS2 peak lists with explicit precursorMz metadata.
- The goal is conversion from Python matchms.Spectrum objects back to R — use SpectriPy's dedicated conversion functions rather than manual construction.

## Inputs

- Paired m/z and intensity peak lists per spectrum (numeric vectors)
- Spectrum-level metadata: msLevel (2L), precursorMz (e.g., 195.0877 for Caffeine, 170.0924 for 1-Methylhistidine), compound name, optional HMDB id
- R DataFrame conforming to Spectra's MsBackendDataFrame expectations

## Outputs

- Spectra::Spectra object containing MS2 spectra
- Combined Spectra collection via c(caf, mhd) ready for compareSpectra()/compareSpectriPy()

## How to apply

Assemble a DataFrame whose columns carry spectrum-level metadata (msLevel = c(2L, 2L), precursorMz, name, optional id such as HMDB0000001) and whose mz and intensity columns are lists of numeric vectors with matching lengths per spectrum. Pass the DataFrame to Spectra::Spectra() to obtain a Spectra object; combine multiple compounds (e.g., Caffeine and 1-Methylhistidine) into one collection with c(caf, mhd). Keep peak lists consistent: each spectrum's m/z and intensity vectors must have equal length, and precursor m/z values should be repeated for replicate spectra of the same compound. These in-memory objects are then usable both with Spectra's built-in compareSpectra() and, after automatic conversion via reticulate, with compareSpectriPy() against matchms scoring functions such as CosineHungarian or CosineGreedy.

## Related tools

- **Spectra** (Defines the Spectra::Spectra container and MsBackendDataFrame used to hold the peak-list-derived MS2 spectra; provides compareSpectra() for the built-in scoring step that consumes these objects) — https://github.com/RforMassSpectrometry/Spectra
- **SpectriPy** (Converts the constructed Spectra objects to matchms.Spectrum objects so matchms scoring functions (e.g., CosineHungarian, CosineGreedy) can be applied from R) — https://github.com/rformassspectrometry/SpectriPy

## Examples

```
caf <- Spectra(DataFrame(msLevel = c(2L, 2L), name = "Caffeine", precursorMz = c(195.0877, 195.0877))); res <- compareSpectriPy(c(caf, mhd), caf, param = CosineGreedy(tolerance = 0.05))
```

## Evaluation signals

- Each spectrum's mz and intensity list entries have identical vector lengths (e.g., Caffeine spectrum 1: 4 peaks; spectrum 2: 5 peaks).
- All spectra report msLevel = 2L and non-missing precursorMz matching the compound (195.0877 for Caffeine, 170.0924 for 1-Methylhistidine).
- Combining collections with c(caf, mhd) yields a Spectra object with length equal to the sum of both objects (4 spectra in the vignette case).
- compareSpectra(all, caf) on the constructed objects returns a 4x2 numeric similarity matrix with values in [0, 1] and caffeine self-similarity entries equal to 1.
- The same Spectra objects convert losslessly to matchms.Spectrum via compareSpectriPy(), reproducing comparable score matrices.

## Limitations

- First compareSpectriPy() call on the constructed spectra is slower because the Python setup has to be initialized.
- SpectriPy requires Python >= 3.12 with matchms 0.31, spectrum_utils 0.3.2, and numpy 2.2.0; if RETICULATE_PYTHON/RETICULATE_PYTHON_ENV variables are set, all required Python libraries must be manually installed.
- Only paired m/z/intensity centroided peak lists are represented; no handling of profile data or file-based backends is covered by this construction path.

## Evidence

- [readme] Create a Spectra object with two MS2 spectra for Caffeine.: "#' Create a Spectra object with two MS2 spectra for Caffeine."
- [readme] DataFrame with msLevel, precursorMz, and list columns of mz and intensity peaks: "caf <- DataFrame(
    msLevel = c(2L, 2L),
    name = "Caffeine",
    precursorMz = c(195.0877, 195.0877)
)"
- [readme] Creating the 1-Methylhistidine spectra with HMDB id and precursor m/z 170.0924: "mhd <- DataFrame(
    msLevel = c(2L, 2L),
    precursorMz = c(170.0924, 170.0924),
    id = c("HMDB0000001", "HMDB0000001"),"
- [readme] Combining Spectra objects and computing pairwise similarities against the caffeine spectra: "all <- c(caf, mhd)
res_r <- compareSpectra(all, caf)"
- [readme] compareSpectriPy returns a numeric matrix of similarities after conversion of the Spectra objects: "As a result `compareSpectriPy()` returns also a numeric matrix of similarities."
- [intro] SpectriPy wraps Python functionality into R functions so converted spectra objects work in one workflow: "By wrapping Python functionality into R
functions, *SpectriPy* allows a
seamless integration of Python libraries into R."
