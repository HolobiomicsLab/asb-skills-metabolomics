---
name: mass-spectrum-peak-count-verification
description: Use when a matrix-only template peak list has been subtracted from an
  MSI peak list (e.g. via mass2adduct subtractPeaks) and you must verify the resulting
  cleaned peak count matches the expected removal before downstream adduct identification.
when_to_use_negative:
- When the goal is adduct identification or co-localization testing rather than verifying
  the peak count after subtraction — use massdiff()/adductMatch()/corrPairsMSI() for
  those steps.
- When the input is an already-cleaned or feature-reduced peak table whose count is
  known a priori — re-running subtractPeaks just to re-verify counts is redundant.
- When the mass accuracy of the data does not support a 5 ppm matching tolerance (e.g.
  lower-resolution instruments) — bin-width/ppm choices must be adjusted and the expected
  removal count recalculated.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: Provides msimat()/msimatreadcsv() to import the peak list, subtractPeaks()
    to remove matrix template peaks within a 5 ppm tolerance, and print/summary of
    the `msimat` object to verify the remaining peak count.
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment in which the mass2adduct package is loaded (library(mass2adduct))
    and the verification steps are executed.
- name: SCiLS
  role: Source of the plain-text CSV peak-list export that serves as the input to
    msimat()/msimatreadcsv() before subtraction and count verification.
- name: MSiReader
  role: Alternative exporter of the plain-text CSV MSI peak list consumed by msimat()
    for the subtraction-and-verification workflow.
- name: Cardinal
  role: If the data are processed in Cardinal, MSProcessedImagingExperiment/MSContinuousImagingExperiment
    objects are converted to `msimat` via cardinal2msimat() before subtraction and
    count verification.
  repo: http://cardinalmsi.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:627bcefa11bd37ddc37ad475f1bb3cfee4f7d21e6cdb33420aa884e8f71d7fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - imzML
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# mass-spectrum-peak-count-verification

## Summary

Quantitative verification of the peak counts remaining after subtracting matrix-only template peaks from a MALDI-MSI peak list, confirming that the mass2adduct subtractPeaks() step removed the expected number of matrix cluster ions within a defined ppm tolerance.

## When to use

Use when a matrix-only template peak list has been subtracted from an MSI peak list (e.g. via mass2adduct subtractPeaks) and you must verify the resulting cleaned peak count matches the expected removal before downstream adduct identification.

## When NOT to use

- When the goal is adduct identification or co-localization testing rather than verifying the peak count after subtraction — use massdiff()/adductMatch()/corrPairsMSI() for those steps.
- When the input is an already-cleaned or feature-reduced peak table whose count is known a priori — re-running subtractPeaks just to re-verify counts is redundant.
- When the mass accuracy of the data does not support a 5 ppm matching tolerance (e.g. lower-resolution instruments) — bin-width/ppm choices must be adjusted and the expected removal count recalculated.

## Inputs

- Original MALDI-MSI peak list exported as plain-text CSV from SCiLS or MSiReader (e.g. 8208 peaks for dataset #1)
- Matrix-only template peak list (e.g. combined Super-DHB matrix peaks whose subtraction removes 2243 matrix peaks)
- Tolerance parameters for subtractPeaks (ppm1, ppm2 set to 5 ppm)

## Outputs

- Cleaned `msimat` object (or CSV of per-peak m/z and intensity, e.g. cleaned_peak_list.csv) whose peak count is verified against the expected value (e.g. 5965 peaks)
- Summary line from print(msimat) reporting the verified peak count

## How to apply

Load the original MSI peak list (e.g. dataset #1's 8208 peaks) into R as an `msimat` object with mass2adduct's msimat()/msimatreadcsv(), and load the matrix-only template peak list (e.g. combined Super-DHB matrix peaks). Call subtractPeaks(input, vec, ppm1, ppm2) to remove template-matching peaks within the 5 ppm tolerance, retaining indices where available. Summarize the cleaned `msimat` object to count the remaining peaks, then check that the reported count matches the expected invariant (original count minus removed count, e.g. 8208 - 2243 = 5965). Only proceed to adduct identification (massdiff() followed by adductMatch() against the built-in `adducts` data) once the count is verified, so downstream mass-difference and co-localization analyses are computed on the intended cleaned peak set.

## Related tools

- **mass2adduct** (Provides msimat()/msimatreadcsv() to import the peak list, subtractPeaks() to remove matrix template peaks within a 5 ppm tolerance, and print/summary of the `msimat` object to verify the remaining peak count.) — https://github.com/kbseah/mass2adduct
- **R** (Runtime environment in which the mass2adduct package is loaded (library(mass2adduct)) and the verification steps are executed.)
- **SCiLS** (Source of the plain-text CSV peak-list export that serves as the input to msimat()/msimatreadcsv() before subtraction and count verification.)
- **MSiReader** (Alternative exporter of the plain-text CSV MSI peak list consumed by msimat() for the subtraction-and-verification workflow.)
- **Cardinal** (If the data are processed in Cardinal, MSProcessedImagingExperiment/MSContinuousImagingExperiment objects are converted to `msimat` via cardinal2msimat() before subtraction and count verification.) — http://cardinalmsi.org/

## Examples

```
library(mass2adduct); d <- msimat("msi.csv", sep=";"); d.clean <- subtractPeaks(d, template_peaks, ppm1=5, ppm2=5); print(d.clean) # expect 5965 peaks
```

## Evaluation signals

- Count invariant: the cleaned peak list must contain exactly (original peaks − template-removed peaks), e.g. 8208 − 2243 = 5965 for dataset #1.
- print(d_clean) or summary output must report a peak count matching the expected value; any discrepancy flags an incorrect ppm window or template mismatch.
- The cleaned CSV (e.g. cleaned_peak_list.csv) must have one row per surviving peak with m/z and intensity columns, and row count equal to the verified count.
- Re-running subtractPeaks with the same 5 ppm tolerance should reproduce the same removal count (idempotency check).
- Downstream massdiff()/adductMatch() results computed on the cleaned list should reflect only non-matrix peaks; matrix-cluster mass differences (e.g. Super-DHB clusters) should no longer dominate topAdducts() output.

## Limitations

- The verification depends on the template peak list being a faithful matrix-only spectrum; an incomplete template under-removes matrix clusters and yields a too-high cleaned count.
- subtractPeaks matching is bounded by the ppm tolerance (5 ppm in the article's example); too narrow a window misses true matrix matches, too wide a window removes genuine metabolite peaks.
- Large CSV peak lists exported from SCiLS or MSiReader can exceed available memory; the msimunging.pl script and triplet-format `msimat` import are workarounds but still memory-limited.

## Evidence

- [full_text] Cleaned peak count matches article's reported matrix removal: [evidence span withheld — anchor status: not_found]
- [full_text] subtractPeaks with ppm tolerance removes template peaks: [evidence span withheld — anchor status: not_found]
- [readme] Peak-list import via msimat from CSV exports: [evidence span withheld — anchor status: external_doc]
- [readme] msimat summary used to report peak counts: "d <- msimat("msi.csv", sep=";")
class(d) # "msimat"
print(d) # Report a summary"
- [readme] Downstream adduct matching follows on the cleaned list: "The package comes with a small data set of common molecular adducts, called `adducts`"
- [readme] ppm tolerance is a user-set parameter in adduct workflows: [evidence span withheld — anchor status: external_doc]
