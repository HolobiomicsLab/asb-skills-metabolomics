---
name: mass-spectrometry-feature-annotation-evaluation
description: Use when an xsAnnotate/CAMERA peak list (e.g. faahKO or MM39) must be
  checked for correct isotope cluster membership and charge states, or when benchmarking
  a reimplementation of CAMERA's findIsotopes logic.
when_to_use_negative:
- Input is a raw centroided mzML/NetCDF file that has not been peak-picked, grouped
  into pseudospectra, and packaged as an xsAnnotate object
- Goal is adduct or fragment annotation rather than isotope/charge evaluation — CAMERA
  handles those with separate rule-table logic (e.g. grouping adducts, neutral-loss
  rule sets)
- Compound spectra contain only the molecular ion and fragment ions with no isotopologue
  pairs, so no charge state can be deduced from 1.0033/z spacing
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package supplying findIsotopes/findIsotopesWithValidation, the
    pairwise m/z distance rule, and isotope/adduct annotation within compound spectra
  repo: https://github.com/sneumann/CAMERA
- name: xcms
  role: Upstream feature detection and grouping that produces the peak lists / xsAnnotate
    objects evaluated here
- name: R
  role: Implementation language; CAMERA packages for Windows, Mac OS and Linux are
    distributed via the Bioconductor repository
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:a84631b57a86f336852e69659811c9b8526f7084bbac0794748a3f026abf47b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
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

# mass-spectrometry-feature-annotation-evaluation

## Summary

Evaluates LC/MS feature annotation quality — specifically CAMERA's isotope peak detection and charge state deduction within compound spectra — by reconstructing the pairwise m/z-distance isotope rule and scoring agreement against a reference annotation such as the MM39 mixture of 39 known compounds.

## When to use

Use when an xsAnnotate/CAMERA peak list (e.g. faahKO or MM39) must be checked for correct isotope cluster membership and charge states, or when benchmarking a reimplementation of CAMERA's findIsotopes logic.

## When NOT to use

- Input is a raw centroided mzML/NetCDF file that has not been peak-picked, grouped into pseudospectra, and packaged as an xsAnnotate object
- Goal is adduct or fragment annotation rather than isotope/charge evaluation — CAMERA handles those with separate rule-table logic (e.g. grouping adducts, neutral-loss rule sets)
- Compound spectra contain only the molecular ion and fragment ions with no isotopologue pairs, so no charge state can be deduced from 1.0033/z spacing

## Inputs

- MM39 peak list or xsAnnotate object from faahKO/MM39 xcms preprocessing
- CAMERA parameters: maxcharge, maxiso, ppm, mzabs, intval
- CAMERA reference annotation table for MM39 (isotope rank and charge per feature)

## Outputs

- camera_isotope_charge_eval.csv with one row per compound spectrum and columns for isotope cluster membership, deduced charge, and reference match status
- Per-compound-spectrum agreement rates and overall fraction of features whose isotope status and charge state match the reference

## How to apply

Load the MM39 peak list or xsAnnotate object into R with CAMERA from the Bioconductor repository, then build the pairwise m/z distance matrix between all features in each compound spectrum (pseudospectrum). Flag candidate isotope pairs where the m/z difference matches 1.0033/z within the ppm and mzabs tolerances passed to findIsotopes/findIsotopesWithValidation, and additionally require the monoisotopic-to-isotope intensity ratio to fall in the range expected by the isotope model. Group flagged peaks into isotope clusters bounded by the maxiso and maxcharge parameters and deduce charge state z from the observed 1/z isotope spacing. Compare every annotated feature's isotope rank and charge against the CAMERA reference annotation for MM39 and record per-compound-spectrum agreement.

## Related tools

- **CAMERA** (Bioconductor package supplying findIsotopes/findIsotopesWithValidation, the pairwise m/z distance rule, and isotope/adduct annotation within compound spectra) — https://github.com/sneumann/CAMERA
- **xcms** (Upstream feature detection and grouping that produces the peak lists / xsAnnotate objects evaluated here)
- **R** (Implementation language; CAMERA packages for Windows, Mac OS and Linux are distributed via the Bioconductor repository)

## Examples

```
Rscript -e 'library(CAMERA); xs <- groupFWHM(object); xs <- findIsotopes(xs, maxcharge=3, maxiso=5, ppm=10, mzabs=0.015); write.csv(isotopeAnnotation(xs), "camera_isotope_charge_eval.csv")'
```

## Evaluation signals

- Fraction of features whose deduced charge state equals the CAMERA reference charge for MM39 (compare against CAMERA's reported recovery of accurate masses for 89.7%/90.3% of annotatable compounds in positive/negative mode as an order-of-magnitude sanity anchor)
- Per-compound-spectrum agreement rate computed over all pseudospectra in camera_isotope_charge_eval.csv, not just an aggregate
- Every flagged isotope pair's m/z difference must fall within the ppm and mzabs tolerances of 1.0033/z
- Intensity ratio between monoisotopic peak and candidate isotope peak lies inside the isotope model's expected range
- Isotope cluster sizes do not exceed the maxiso limit and inferred charges do not exceed maxcharge

## Limitations

- Missing molecular mass annotations occurred when compound spectra contained too few explained features, when several hypotheses tied in precedence score, or when spectra contained only molecular ion and fragment ions with no further adducts
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- Sample concentration must avoid MS detector saturation, which shifts m/z and retention time centroids and truncates peak profiles

## Evidence

- [results] CAMERA calculates a pairwise m/z distance matrix and detects isotopes as pairs exhibiting an m/z difference of 1.0033/z that also pass an additional intensity ratio check: [evidence span withheld — anchor status: not_found]
