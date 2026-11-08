---
name: chromatographic-peak-fwhm-window-grouping
description: Use when an aligned XCMS feature table (m/z, retention time, apex intensity,
  per-sample intensities) must be partitioned into co-eluting compound spectra as
  the first CAMERA grouping step before isotope/adduct annotation.
when_to_use_negative:
- Features were already annotated by another grouping pass (e.g. correlation-based
  calcCiS/calcCaS) — groupFWHM is the initial step and assumes an unassigned feature
  table.
- Data lack chromatographic separation (direct infusion MS), so retention-time windows
  around peak FWHM are meaningless for co-elution inference.
- The goal is ion-species annotation (adducts, isotopes, fragments) rather than the
  initial co-elution grouping — use findIsotopes/findAdducts after this step instead.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: CAMERA
  role: Hosts the groupFWHM routine that implements this skill; an R/Bioconductor
    package whose xsAnnotate object tracks pseudospectrum assignments.
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Produces the aligned feature table (m/z, retention time, intensities) that
    is the input feature list the grouping iterates over.
- name: faahKO
  role: Example dataset used to exercise the grouping routine and validate the pseudospectrum
    output.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:1b7ff89a126509e8cce4a5fa560a460151a63a2c8e23b343944ab42263ef05d0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  - faahKO
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# chromatographic-peak-fwhm-window-grouping

## Summary

Initial CAMERA compound-spectrum extraction step (groupFWHM) that clusters LC/MS features sharing a retention-time window around each unassigned feature's chromatographic peak FWHM into pseudospectra, iteratively until every feature in the XCMS feature table is assigned.

## When to use

Use when an aligned XCMS feature table (m/z, retention time, apex intensity, per-sample intensities) must be partitioned into co-eluting compound spectra as the first CAMERA grouping step before isotope/adduct annotation.

## When NOT to use

- Features were already annotated by another grouping pass (e.g. correlation-based calcCiS/calcCaS) — groupFWHM is the initial step and assumes an unassigned feature table.
- Data lack chromatographic separation (direct infusion MS), so retention-time windows around peak FWHM are meaningless for co-elution inference.
- The goal is ion-species annotation (adducts, isotopes, fragments) rather than the initial co-elution grouping — use findIsotopes/findAdducts after this step instead.

## Inputs

- XCMS feature table (m/z, retention time, apex retention time, FWHM or peak bounds, apex intensity, per-sample intensities)
- xsAnnotate object created from the XCMS peak table (e.g. from the faahKO example dataset)
- Optional reference grouping, e.g. the MM39 mixture of 39 known compounds with known co-eluting features

## Outputs

- xsAnnotate object annotated with numbered compound spectra (pseudospectra) per feature
- getPeaklist-derived peak table including compound-spectrum (pseudospectrum) assignment per feature
- Agreement metrics comparing FWHM-based grouping to a reference compound grouping (e.g. against the MM39 standard mixture)

## How to apply

Load the XCMS feature table into an xsAnnotate object and run groupFWHM iteratively: pick the most intense feature not yet assigned to any compound spectrum, center a retention-time window on its chromatographic apex, and collect all remaining unassigned features whose retention times fall within a window equal to 60% of that feature's chromatographic peak FWHM. Assign the collected features to the current pseudospectrum and repeat with the next most intense unassigned feature until no feature remains unassigned, yielding numbered pseudospectra in the xsAnnotate object. The rationale is that features derived from the same analyte co-elute, so an FWHM-scaled window is a first, cheap approximation of the analyte's chromatographic footprint before higher-cost correlation (calcCiS/calcCaS) or m/z-difference methods refine the grouping. Validate the result with getPeaklist, which returns the per-feature pseudospectrum index alongside intensities, and benchmark agreement against a reference where each compound's co-eluting features are known (e.g. the MM39 mixture of 39 known compounds spanning 161–822 Da).

## Related tools

- **CAMERA** (Hosts the groupFWHM routine that implements this skill; an R/Bioconductor package whose xsAnnotate object tracks pseudospectrum assignments.) — https://github.com/sneumann/CAMERA
- **XCMS** (Produces the aligned feature table (m/z, retention time, intensities) that is the input feature list the grouping iterates over.)
- **faahKO** (Example dataset used to exercise the grouping routine and validate the pseudospectrum output.)

## Examples

```
library(CAMERA); xs <- xsAnnotate(faahKO::faahKO, samplegroups=c("WT","KO")); xs <- groupFWHM(xs, ppm=10, mw=0.5); pl <- getPeaklist(xs)
```

## Evaluation signals

- Every feature in the input table ends up assigned to exactly one compound spectrum (no features remain unassigned after the iteration terminates).
- Compound-spectrum count and membership reproducible when re-run on the same xsAnnotate object with identical parameters (60% FWHM default).
- Assignment agreement against a reference grouping (MM39 standard with known co-eluting features) exceeds a benchmark threshold consistent with CAMERA's reported accuracy (e.g. ≥ 89.7% accurate-mass recovery).
- getPeaklist output schema contains the expected columns: m/z, retention time, intensity, sample intensities, and pseudospectrum index.
- Features assigned to the same pseudospectrum have apex retention times within one FWHM-scaled window of each other, consistent with co-elution.

## Limitations

- Coarse first approximation: features may be grouped into the same pseudospectrum merely because their retention times overlap within an FWHM-scaled window, even if they derive from different analytes that happen to co-elute; correlation- or m/z-based refinement (calcCiS, calcCaS, calcPC) is needed to tighten the grouping.
- Window size is tied to a single feature's FWHM (default 60%), so features with unusually narrow or broad peak shapes can under- or over-include neighbors; results depend on accurate chromatographic peak bounds from the upstream feature detector.
- MS detector saturation distorts m/z and retention time centroids and truncates peak profiles, which corrupts the FWHM estimate the window is built from — sample concentration must avoid saturation.
- Accurate-mass extraction can still fail downstream if the resulting compound spectra contain too few explained features, tied precedence scores, or only molecular and fragment ions without further adducts.

## Evidence

- [methods] groupFWHM iteratively picks the most intense unassigned feature and collects unassigned features within a retention-time window equal to 60% of that feature's FWHM: [evidence span withheld — anchor status: not_found]
