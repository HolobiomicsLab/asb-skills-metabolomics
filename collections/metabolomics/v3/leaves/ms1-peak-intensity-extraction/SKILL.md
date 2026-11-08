---
name: ms1-peak-intensity-extraction
description: Use when high-resolution MS/MS or MS1 data need precursor and isotope
  peak abundances extracted after preprocessing, as input to isotope pattern-based
  molecular formula deduction (e.g. SIRIUS).
when_to_use_negative:
- The goal is fragmentation tree computation or fingerprint-based structure search
  from MS2 spectra rather than formula ranking from isotope pattern intensities
- Only low-resolution MS1 data are available — SIRIUS deduces formulas by ranking
  isotope patterns from mass spectra of high resolution
- The input is already a scored formula candidate table; extraction would duplicate
  an upstream preprocessing step
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS 4
  role: Provides the isotope pattern analysis pipeline into which the extracted MS1
    peak intensities are fed for molecular formula ranking
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Java-based framework whose preprocessed peak lists (masses and abundances)
    are the direct output consumed by this extraction step
  repo: https://github.com/sirius-ms/sirius
- name: demo-data
  role: SIRIUS demo dataset supplying per-compound MS1 isotope peak intensities for
    validating the extraction workflow
  repo: https://github.com/sirius-ms/sirius/raw/refs/heads/stable/data/demo-data.zip
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:789f7c8611afdb686db33b16088faaf8a06cd0c9d9acad0e071bd5c2eaee5eed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  - demo-data
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ms1-peak-intensity-extraction

## Summary

Extracts measured MS1 isotope peak intensities (peak masses and abundances) from preprocessed high-resolution mass spectra so they can be scored against theoretical isotope patterns for molecular formula ranking in SIRIUS.

## When to use

Use when high-resolution MS/MS or MS1 data need precursor and isotope peak abundances extracted after preprocessing, as input to isotope pattern-based molecular formula deduction (e.g. SIRIUS).

## When NOT to use

- The goal is fragmentation tree computation or fingerprint-based structure search from MS2 spectra rather than formula ranking from isotope pattern intensities
- Only low-resolution MS1 data are available — SIRIUS deduces formulas by ranking isotope patterns from mass spectra of high resolution
- The input is already a scored formula candidate table; extraction would duplicate an upstream preprocessing step

## Inputs

- Preprocessed high-resolution MS1 mass spectrum as a list of peaks (peak masses and abundances)
- MS/MS input files as accepted by SIRIUS (e.g. the demo-data.zip dataset of LC-MS/MS records)
- Candidate molecular formulas with theoretical isotope patterns for intensity-pair alignment

## Outputs

- Per-compound list of measured MS1 isotope peak masses and abundances
- Measured/theoretical intensity pairs used for isotope pattern likelihood scoring
- Ranked candidate molecular formulas with total log-odds isotope pattern scores (e.g. isotope_pattern_scores.csv)

## How to apply

After mass spectrometer preprocessing, the measured data is a list of peaks with the masses of the sample molecules and their abundances; parse per-compound records (e.g. from the SIRIUS demo-data.zip dataset) to pull the MS1 isotope peak masses and intensities of the parent ion's isotope envelope. If MS1 data is available, use the parent peak from MS1 as the anchor for decomposition, then collect the isotope peaks that follow it. Compute theoretical isotope patterns for each candidate molecular formula by convolving per-element natural isotope abundances so each measured/theoretical intensity pair can be aligned peak-by-peak. These extracted intensities feed the SIRIUS 4 intensity error model Y = x + D + E, with relative noise D ~ N(0, x^2*σ_rel^2) and absolute noise E ~ N(0, σ_abs^2) (defaults σ_abs = 0.01, σ_rel = 0.08), whose closed-form maximum likelihood yields per-peak log-odds scores summed across the isotope pattern. Correctness is judged by verifying that every compound's extracted peak list yields finite log-odds scores for all candidate formulas and that the theoretical pattern's peak masses match the measured envelope within high-resolution mass accuracy.

## Related tools

- **SIRIUS 4** (Provides the isotope pattern analysis pipeline into which the extracted MS1 peak intensities are fed for molecular formula ranking) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Java-based framework whose preprocessed peak lists (masses and abundances) are the direct output consumed by this extraction step) — https://github.com/sirius-ms/sirius
- **demo-data** (SIRIUS demo dataset supplying per-compound MS1 isotope peak intensities for validating the extraction workflow) — https://github.com/sirius-ms/sirius/raw/refs/heads/stable/data/demo-data.zip

## Examples

```
sirius --input demo-data.zip --output isotope_pattern_scores --maxmz 500 formula
```

## Evaluation signals

- Every compound in the demo dataset receives finite log-odds scores for all candidate formulas in the output table
- The closed-form likelihood implementation numerically reproduces equation (2) on synthetic measured/theoretical intensity pairs
- Extracted MS1 peak masses align with the theoretical isotope envelope of candidate formulas within high-resolution mass accuracy
- Summed per-peak log-odds ranks the correct molecular formula top-scored for compounds with known ground truth in the demo data

## Limitations

- Isotope pattern selection has had bugs in past SIRIUS releases; the isotope pattern analysis module was rewritten and is only 'hopefully more stable than before'
- The implementation swaps x and y so relative noise depends on the observed rather than the candidate intensity, a deviation from the paper's model that affects strict reproduction of equation (2)
- If MS1 data is not available, the intensity-based isotope pattern score cannot be computed and decomposition relies on the parent peak alone
- Support for in-source fragmentations and adducts is experimental, which can contaminate the extracted isotope envelope

## Evidence

- [methods] Preprocessing peak lists: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [readme] Peak list structure: "After preprocessing, the output of a mass spectrometer is a list of peaks which corresponds to the masses of the sample molecules and their abundance."
- [discussion] MS1 parent peak handling: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the"
- [readme] Demo dataset: "- [Demo data](https://github.com/sirius-ms/sirius/raw/refs/heads/stable/data/demo-data.zip?download=)"
- [discussion] Isotope pattern analysis rewrite: "isotope pattern analysis is now rewritten and hopefully more stable than before"
