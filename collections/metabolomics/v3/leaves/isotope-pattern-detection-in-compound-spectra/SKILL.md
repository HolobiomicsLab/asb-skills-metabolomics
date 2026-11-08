---
name: isotope-pattern-detection-in-compound-spectra
description: Use when annotating LC/MS peak lists (e.g. xsAnnotate objects from faahKO/MM39
  processing) and you must identify isotope clusters and charge states within each
  pseudospectrum rather than treating isotope peaks as separate compounds.
when_to_use_negative:
- Input features have not been grouped into compound spectra (pseudospectra) yet —
  isotope detection within CAMERA operates per pseudospectrum, not on an ungrouped
  peak table.
- The goal is adduct or fragment annotation rather than isotope/charge assignment
  — CAMERA handles those via the dynamic rule table separately.
- Only the molecular ion and fragment ions are present with no isotopic siblings —
  as the article notes, such spectra cannot always be annotated directly.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CAMERA
  role: Bioconductor package whose findIsotopes/findIsotopesWithValidation perform
    the pairwise m/z distance matrix, isotope pair detection, and charge deduction
    within compound spectra
  repo: https://github.com/sneumann/CAMERA
- name: xcms
  role: Upstream feature detection producing the peak lists/xsAnnotate objects that
    CAMERA annotates
- name: R
  role: Implementation language; CAMERA packages for Windows (32/64 bit), Mac OS and
    Linux come from the Bioconductor repository
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:8fdd5e6c5ea6cf11bab2d1bcb647776250b349ed6f28d1547c97703ab2354d81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# isotope-pattern-detection-in-compound-spectra

## Summary

Detects isotopic peaks within CAMERA compound spectra (pseudospectra) by computing a pairwise m/z distance matrix and flagging pairs whose m/z difference is close to 1.0033/z within ppm and mzabs tolerances plus an intensity-ratio check, enabling deduction of charge states.

## When to use

Use when annotating LC/MS peak lists (e.g. xsAnnotate objects from faahKO/MM39 processing) and you must identify isotope clusters and charge states within each pseudospectrum rather than treating isotope peaks as separate compounds.

## When NOT to use

- Input features have not been grouped into compound spectra (pseudospectra) yet — isotope detection within CAMERA operates per pseudospectrum, not on an ungrouped peak table.
- The goal is adduct or fragment annotation rather than isotope/charge assignment — CAMERA handles those via the dynamic rule table separately.
- Only the molecular ion and fragment ions are present with no isotopic siblings — as the article notes, such spectra cannot always be annotated directly.

## Inputs

- xsAnnotate object or MM39 peak list from CAMERA/xcms preprocessing
- Rule set / distance parameters: ppm tolerance, mzabs tolerance, maxiso, maxcharge, intval

## Outputs

- Isotope/charge annotation table mapping each feature to compound spectrum, isotope rank, and charge state
- Per-compound-spectrum agreement rates versus reference annotation (e.g. camera_isotope_charge_eval.csv)

## How to apply

Load the MM39 peak list or xsAnnotate object into R with CAMERA from Bioconductor, then build the pairwise m/z distance matrix between all features in each compound spectrum. Flag candidate isotope pairs where the m/z difference is close to 1.0033/z within the ppm and mzabs tolerances, and additionally verify that the intensity ratio between the monoisotopic peak and the candidate isotope falls within the range expected by the isotope model. Group flagged peaks into isotope clusters using the maxiso and maxcharge parameters of findIsotopes/findIsotopesWithValidation, and deduce charge state z from the observed spacing of 1/z. Emit an annotation table mapping each feature to its compound spectrum, isotope rank, and charge state, and score agreement against a reference annotation per compound spectrum.

## Related tools

- **CAMERA** (Bioconductor package whose findIsotopes/findIsotopesWithValidation perform the pairwise m/z distance matrix, isotope pair detection, and charge deduction within compound spectra) — https://github.com/sneumann/CAMERA
- **xcms** (Upstream feature detection producing the peak lists/xsAnnotate objects that CAMERA annotates)
- **R** (Implementation language; CAMERA packages for Windows (32/64 bit), Mac OS and Linux come from the Bioconductor repository)

## Examples

```
Rscript -e 'library(CAMERA); xs <- groupFWHM(xsa <- xsAnnotate(faahKOxs)); xs <- findIsotopes(xs, ppm=10, mzabs=0.01, maxcharge=2, maxiso=4)'
```

## Evaluation signals

- Fraction of features whose isotope status and deduced charge state match the CAMERA reference annotation for MM39.
- Per-compound-spectrum agreement rates reported in camera_isotope_charge_eval.csv with columns for isotope cluster membership, deduced charge, and reference match status.
- Flagged isotope pairs satisfy both criteria: m/z difference ≈ 1.0033/z within ppm/mzabs tolerance AND intensity ratio within the isotope model's expected range.
- Deduced charge spacing: consecutive isotope members differ by approximately 1/z in m/z, not by 1.0033 Da for z > 1.
- Every annotated feature is mapped to exactly one compound spectrum with a defined isotope rank.

## Limitations

- Isotope detection depends on pseudospectrum quality: compound spectra with too few explained features yield missing annotations.
- Intensity-ratio validation can reject true isotope pairs when peaks are near the intensity threshold or affected by detector saturation (which causes m/z and RT centroid shifts).
- maxcharge bounds the deducible charge states; higher-charge isotopologues spaced by less than 1.0033 Da require adjusting maxcharge.

## Evidence

- [methods] Within each compound spectrum, CAMERA calculates a pairwise m/z distance matrix and detects isotopes as pairs exhibiting an m/z difference of 1.0033/z: [evidence span withheld — anchor status: not_found]
