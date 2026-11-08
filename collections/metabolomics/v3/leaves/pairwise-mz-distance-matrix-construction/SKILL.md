---
name: pairwise-mz-distance-matrix-construction
description: Use when annotating isotope peaks and charge states within grouped LC/MS
  compound spectra (pseudospectra) from an xsAnnotate/CAMERA peak table, e.g. features
  from faahKO or MM39-style mixtures where spacing near 1.0033/z must be identified.
when_to_use_negative:
- You need adduct, fragment, or quasi-molecular-ion annotation from the CAMERA rule
  table — this skill covers only the isotope/charge-spacing step, not the full dynamic
  rule-table annotation.
- Features have not been grouped into compound spectra yet; a pairwise m/z distance
  matrix over ungrouped features across the whole run is not the CAMERA procedure.
- The compound spectrum contains only the molecular ion and fragment ions with no
  further adducts/isotopes — CAMERA reports such spectra cannot be annotated directly
  unless the neutral loss is added to the rule set.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package whose findIsotopes/findIsotopesWithValidation routines
    implement the m/z-distance isotope screening (ppm, mzabs, maxcharge, intval parameters)
    and isotope-cluster formation
  repo: https://github.com/sneumann/CAMERA
- name: Bioconductor
  role: Repository from which the CAMERA package is installed for Windows, Mac OS
    and Linux
- name: R
  role: Implementation language in which the pairwise m/z distance matrix and isotope/charge
    annotation are computed
- name: xcms
  role: Produces the feature peak lists and xsAnnotate-derived grouping that feed
    the per-pseudospectrum m/z distance calculation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:a6108ec5dcd04655b733b3992885d97de274ce1e85b56c020cf012f6359dca27
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - Bioconductor
  - CAMERA
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pairwise-mz-distance-matrix-construction

## Summary

Reconstructs CAMERA's in-pseudospectrum isotope detection by building a pairwise m/z distance matrix over all features in each compound spectrum, flagging pairs whose difference matches 1.0033/z within ppm and mzabs tolerances, checking intensity ratios, and deducing charge states from the observed 1/z spacing.

## When to use

Use when annotating isotope peaks and charge states within grouped LC/MS compound spectra (pseudospectra) from an xsAnnotate/CAMERA peak table, e.g. features from faahKO or MM39-style mixtures where spacing near 1.0033/z must be identified.

## When NOT to use

- You need adduct, fragment, or quasi-molecular-ion annotation from the CAMERA rule table — this skill covers only the isotope/charge-spacing step, not the full dynamic rule-table annotation.
- Features have not been grouped into compound spectra yet; a pairwise m/z distance matrix over ungrouped features across the whole run is not the CAMERA procedure.
- The compound spectrum contains only the molecular ion and fragment ions with no further adducts/isotopes — CAMERA reports such spectra cannot be annotated directly unless the neutral loss is added to the rule set.

## Inputs

- MM39 peak list (feature table with m/z, retention time, intensity)
- xsAnnotate object from faahKO/MM39 xcms processing
- CAMERA ruleSet / isotope distance parameters (ppm, mzabs, maxcharge, maxiso)

## Outputs

- Per-pseudospectrum pairwise m/z distance matrix
- Flagged isotope-pair list with intensity-ratio validation status
- Isotope cluster annotations with deduced charge state z
- camera_isotope_charge_eval.csv (one row per compound spectrum: isotope cluster membership, deduced charge, reference match status)

## How to apply

Load the MM39 peak list or the xsAnnotate object from the faahKO/MM39 xcms processing into R with CAMERA installed from Bioconductor. For each compound spectrum, compute the pairwise m/z difference matrix between all features rather than comparing peaks pairwise ad hoc — this mirrors CAMERA's internal grouping structure before rule matching. Flag candidate isotope pairs where the m/z difference is close to 1.0033/z within the ppm and mzabs tolerances (the knobs exposed by findIsotopes/findIsotopesWithValidation via maxcharge, ppm, mzabs, intval), and additionally verify that the intensity ratio between the monoisotopic peak and the candidate isotope peak falls in the range expected for the isotope model. Group flagged isotope peaks into isotope clusters using the maxiso/maxcharge parameters, deriving the charge state z from the observed spacing (1/z). Emit an annotation table mapping each feature to its compound spectrum, isotope rank, and charge state, then score against the CAMERA reference annotation for MM39 per compound spectrum.

## Related tools

- **CAMERA** (Bioconductor package whose findIsotopes/findIsotopesWithValidation routines implement the m/z-distance isotope screening (ppm, mzabs, maxcharge, intval parameters) and isotope-cluster formation) — https://github.com/sneumann/CAMERA
- **Bioconductor** (Repository from which the CAMERA package is installed for Windows, Mac OS and Linux)
- **R** (Implementation language in which the pairwise m/z distance matrix and isotope/charge annotation are computed)
- **xcms** (Produces the feature peak lists and xsAnnotate-derived grouping that feed the per-pseudospectrum m/z distance calculation)

## Examples

```
iso <- findIsotopes(xsa, maxcharge=3, ppm=10, mzabs=0.015, intval="into")
```

## Evaluation signals

- Fraction of features whose isotope status and charge state agree with the CAMERA reference annotation for MM39
- Per-compound-spectrum agreement rates reported in camera_isotope_charge_eval.csv (one row per compound spectrum with isotope cluster membership, deduced charge, and reference match status)
- Every flagged isotope pair satisfies Δm/z ≈ 1.0033/z within the configured ppm and mzabs tolerances
- Deduced charge states are consistent with the observed isotope spacing (multiples of 1/z Da ≈ 1.0033)
- findIsotopesWithValidation is run with explicit maxcharge, ppm, mzabs, and intval parameters, not defaults left implicit

## Limitations

- Detector saturation shifts m/z and retention time centroids of features and truncates peak profiles, corrupting the distance matrix; samples must be concentrated to avoid saturation.
- Compound spectra containing too few explained features, tied precedence scores, or only the molecular ion and fragment ions yield missing annotations unless the neutral loss is added to the rule set.
- CAMERA deliberately avoids ad-hoc heuristics such as assuming the most intense feature is the [M+H]+ ion, so intensity alone cannot substitute for the distance/ratio rules.

## Evidence
