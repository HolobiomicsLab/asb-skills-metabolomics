---
name: intensity-ratio-based-isotope-filtering
description: Use when annotating LC/MS pseudospectra where isotope peaks must be flagged
  and charge states deduced from isotope spacing, and candidate isotope pairs need
  an intensity-ratio plausibility check.
when_to_use_negative:
- Spectra containing only the molecular ion and fragment ions with no isotope or adduct
  peaks — CAMERA cannot annotate these directly unless the neutral loss is added to
  the rule set
- Datasets where detector saturation has shifted m/z centroids or truncated peak profiles,
  since the intensity-ratio check relies on reliable peak intensities
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package implementing the pairwise m/z distance matrix, isotope-pair
    detection with intensity-ratio validation, and charge-state deduction
  repo: https://github.com/sneumann/CAMERA
- name: xcms
  role: Upstream feature detection producing the peak lists / xsAnnotate objects on
    which isotope filtering runs
- name: Bioconductor
  role: Distribution channel from which the CAMERA package for Windows, Mac OS and
    Linux is installed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:0b15568ec1ec79e72e90381f3d603d887356b8e0c972b08acde2832e670803a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - Bioconductor
  - CAMERA
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# intensity-ratio-based-isotope-filtering

## Summary

Detect isotopic peak pairs and deduce charge states within CAMERA compound spectra by combining the m/z spacing rule (1.0033/z) with an intensity-ratio check between monoisotopic and isotope peaks, using findIsotopes/maxcharge parameters.

## When to use

Use when annotating LC/MS pseudospectra where isotope peaks must be flagged and charge states deduced from isotope spacing, and candidate isotope pairs need an intensity-ratio plausibility check.

## When NOT to use

- Spectra containing only the molecular ion and fragment ions with no isotope or adduct peaks — CAMERA cannot annotate these directly unless the neutral loss is added to the rule set
- Datasets where detector saturation has shifted m/z centroids or truncated peak profiles, since the intensity-ratio check relies on reliable peak intensities

## Inputs

- xsAnnotate object or MM39 peak list from CAMERA/xcms processing (faahKO/MM39 workflow)
- CAMERA ruleSet/isotope parameter set: ppm tolerance, mzabs tolerance, maxcharge, maxiso, intval

## Outputs

- Isotope/charge annotation table mapping each annotated feature to its compound spectrum, isotope rank, and charge state
- Per-compound-spectrum evaluation table camera_isotope_charge_eval.csv with isotope cluster membership, deduced charge, and reference match status

## How to apply

Load the MM39 peak list or the faahKO xsAnnotate object into R with CAMERA from the Bioconductor repository. Within each compound spectrum, build the pairwise m/z distance matrix and flag candidate isotope pairs whose m/z difference is close to 1.0033/z within the ppm and mzabs tolerances. Additionally check that the intensity ratio between the monoisotopic peak and the candidate isotope peak falls within the range expected for the isotope model — this ratio check filters false isotope calls. Group flagged isotope peaks into isotope clusters using the maxiso/maxcharge parameters of findIsotopes (or findIsotopesWithValidation), and deduce the charge state z from the observed 1/z isotope spacing.

## Related tools

- **CAMERA** (Bioconductor package implementing the pairwise m/z distance matrix, isotope-pair detection with intensity-ratio validation, and charge-state deduction) — https://github.com/sneumann/CAMERA
- **xcms** (Upstream feature detection producing the peak lists / xsAnnotate objects on which isotope filtering runs)
- **Bioconductor** (Distribution channel from which the CAMERA package for Windows, Mac OS and Linux is installed)

## Examples

```
R: library(CAMERA); xs <- xsAnnotate(mm39_peaks); iso <- findIsotopesWithValidation(xs, maxcharge=3, ppm=5, mzabs=0.01, intval="into", validateIsotopePatterns=TRUE)
```

## Evaluation signals

- Fraction of features whose isotope status and charge state agree with the CAMERA reference annotation for MM39
- Per-compound-spectrum agreement rates reported in camera_isotope_charge_eval.csv
- All flagged isotope pairs satisfy both the 1.0033/z spacing (within ppm and mzabs tolerances) and the expected intensity-ratio range
- Each annotated feature is mapped to exactly one compound spectrum, isotope rank, and charge state

## Limitations

- Isotope pairs failing the intensity-ratio model check are excluded, so genuine isotopes with atypical abundances relative to the isotope model may be missed
- Compound spectra with too few explained features, or only molecular ion and fragment ions, cannot be fully annotated
- CAMERA deliberately avoids ad-hoc heuristics such as assuming the most intense feature is the [M+H]+ ion, so annotation depends on the rule table rather than intensity ranking alone

## Evidence

- [results] CAMERA calculates a pairwise m/z distance matrix and detects isotopes as pairs exhibiting an m/z difference of 1.0033/z that also pass an additional intensity ratio check: [evidence span withheld — anchor status: not_found]
