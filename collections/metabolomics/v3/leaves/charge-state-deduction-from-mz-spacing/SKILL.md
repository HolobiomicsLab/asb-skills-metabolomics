---
name: charge-state-deduction-from-mz-spacing
description: Use when you need to label which LC/MS peaks in a grouped compound spectrum
  (pseudospectrum) are isotope peaks and deduce the charge state z from their 1.0033/z
  spacing, e.g. on an xsAnnotate object before adduct annotation.
when_to_use_negative:
- The input is an ungrouped feature list not yet partitioned into CAMERA compound
  spectra/pseudospectra — grouping must precede isotope detection
- Compounds whose spectra contain only a molecular ion and fragment ions with no further
  adducts/isotopes — spacing evidence is insufficient and direct annotation is not
  possible unless the neutral loss is added to the rule set
- Data affected by MS detector saturation, which shifts m/z centroids and breaks the
  1.0033/z spacing check
tools:
- name: CAMERA
  role: Bioconductor package supplying findIsotopes/findIsotopesWithValidation, the
    pairwise m/z distance matrix, and the intensity-ratio isotope validation
  repo: https://github.com/sneumann/CAMERA
- name: R
  role: Implementation language; CAMERA is installed from the Bioconductor repository
- name: xcms
  role: Upstream feature detection producing the peak lists fed into CAMERA's xsAnnotate
    grouping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:f23da1d784189e568ded56fbab9950b1c811d50d5dbbc1b1b26b096091f9407c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - xcms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# charge-state-deduction-from-mz-spacing

## Summary

Reproduces CAMERA's isotope peak detection and charge-state deduction: within each pseudospectrum, pairwise m/z distances are screened for isotope spacing near 1.0033/z with an intensity-ratio validation, yielding per-feature isotope and charge annotations.

## When to use

Use when you need to label which LC/MS peaks in a grouped compound spectrum (pseudospectrum) are isotope peaks and deduce the charge state z from their 1.0033/z spacing, e.g. on an xsAnnotate object before adduct annotation.

## When NOT to use

- The input is an ungrouped feature list not yet partitioned into CAMERA compound spectra/pseudospectra — grouping must precede isotope detection
- Compounds whose spectra contain only a molecular ion and fragment ions with no further adducts/isotopes — spacing evidence is insufficient and direct annotation is not possible unless the neutral loss is added to the rule set
- Data affected by MS detector saturation, which shifts m/z centroids and breaks the 1.0033/z spacing check

## Inputs

- xsAnnotate object or peak list (e.g. MM39/faahKO peak list) processed by CAMERA
- ppm and mzabs tolerance parameters for m/z-distance screening
- maxiso, maxcharge, and intval parameters for findIsotopes/findIsotopesWithValidation

## Outputs

- Per-compound-spectrum isotope/charge annotation table mapping every annotated feature to its compound spectrum, isotope rank, and charge state
- camera_isotope_charge_eval.csv with one row per compound spectrum (isotope cluster membership, deduced charge, reference-match status)

## How to apply

For each compound spectrum built by CAMERA (e.g. from the faahKO/MM39 xsAnnotate object), compute the pairwise m/z distance matrix between all features and flag candidate isotope pairs whose difference is close to 1.0033/z within the ppm and mzabs tolerances. Additionally require the monoisotopic-to-isotope intensity ratio to fall in the range expected by the isotope model — CAMERA deliberately avoids ad-hoc heuristics like assuming the most intense feature is the [M+H]+ ion. Group flagged isotope peaks into isotope clusters per compound spectrum using the maxiso/maxcharge parameters of findIsotopes/findIsotopesWithValidation, and read the charge state z directly from the observed spacing (1/z). Emit an annotation table mapping each feature to its compound spectrum, isotope rank, and charge; validateIsotopePatterns can cross-check clusters, optionally against a database. Compare the result to the CAMERA reference annotation for MM39 to quantify per-pseudospectrum agreement before writing the evaluation CSV.

## Related tools

- **CAMERA** (Bioconductor package supplying findIsotopes/findIsotopesWithValidation, the pairwise m/z distance matrix, and the intensity-ratio isotope validation) — https://github.com/sneumann/CAMERA
- **R** (Implementation language; CAMERA is installed from the Bioconductor repository)
- **xcms** (Upstream feature detection producing the peak lists fed into CAMERA's xsAnnotate grouping)

## Examples

```
findIsotopesWithValidation(object, maxcharge, ppm, mzabs, intval, validateIsotopePatterns, database)
```

## Evaluation signals

- Every flagged isotope pair differs in m/z by ~1.0033/z within the configured ppm and mzabs tolerances
- Deduced charge states within each isotope cluster are consistent with the observed spacing of cluster members
- Feature-level isotope status and charge agree with the CAMERA reference annotation for MM39 (report the fraction of agreeing features)
- camera_isotope_charge_eval.csv contains one row per compound spectrum with the columns isotope-cluster membership, deduced charge, and reference-match status

## Limitations

- Missing annotations arise when compound spectra contain too few explained features, hypotheses tie in precedence score, or spectra hold only molecular and fragment ions
- Compounds with spectra containing only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- MS detector saturation shifts m/z and retention time centroids and truncates peak profiles, corrupting spacing- and ratio-based checks
- Isotope-pair elimination thresholds (e.g. the 1000-count intensity cutoff) can remove genuine low-intensity isotope peaks

## Evidence
