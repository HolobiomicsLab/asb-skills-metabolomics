---
name: isotope-peak-annotation-lcms
description: Use when an LC/MS peak list needs grouping of same-analyte features and
  ion-species annotation (isotopes/adducts) to yield quasi-molecular ions for compound
  identification.
when_to_use_negative:
- The input is already a fully annotated feature table with isotope/adduct labels;
  applying findIsotopes/findAdducts would duplicate or override existing annotations.
- Only a single feature (not a grouped pseudo-spectrum) is available and no isotope/adduct
  context exists, so mass proposal would be unreliable.
- The goal is purely statistical differential analysis on the original feature list
  without reducing feature-list redundancy; grouping and annotation are unnecessary.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: R/Bioconductor package that implements findIsotopes, groupFWHM, groupCorr
    and findAdducts to group and annotate isotope/adduct peaks and propose accurate
    masses
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream preprocessing framework that produces the peak list fed into xsAnnotate
    for annotation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:e2a03a8a298e28be885373c795fa8711f61b8dfd126b40db75805f1577ede823
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  - findAdducts
  - findIsotopes
  - getPeaklist
  - groupCorr
  - groupFWHM
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# isotope-peak-annotation-lcms

## Summary

Annotate isotope and adduct peaks within co-eluting compound spectra using CAMERA's findIsotopes/groupFWHM/groupCorr/findAdducts workflow, then propose accurate compound masses.

## When to use

Use when an LC/MS peak list needs grouping of same-analyte features and ion-species annotation (isotopes/adducts) to yield quasi-molecular ions for compound identification.

## When NOT to use

- The input is already a fully annotated feature table with isotope/adduct labels; applying findIsotopes/findAdducts would duplicate or override existing annotations.
- Only a single feature (not a grouped pseudo-spectrum) is available and no isotope/adduct context exists, so mass proposal would be unreliable.
- The goal is purely statistical differential analysis on the original feature list without reducing feature-list redundancy; grouping and annotation are unnecessary.

## Inputs

- XCMS peak list (features with m/z and retention time) from LC/MS runs in positive and/or negative ionization mode
- MM39 reference compound list with sum formulas, molar masses and PubChem IDs (for accuracy evaluation)
- xsAnnotate object built from the peak list

## Outputs

- Annotated peak list with isotope/adduct annotations and proposed accurate compound masses (via getPeaklist)
- Compound spectra (pseudo-spectra) grouping features derived from the same analyte
- Accurate-mass extraction rate per ion mode (e.g. ≈89.7% positive, ≈90.3% negative on MM39)

## How to apply

Preprocess LC/MS data with XCMS to produce a peak list of detected features, then build an xsAnnotate object and run the CAMERA workflow: groupFWHM groups features into pseudo-spectra by co-elution and Gaussian peak-shape similarity; findIsotopes annotates isotope peaks within each pseudo-spectrum; groupCorr refines grouping using EIC intensity correlation across samples; findAdducts labels adduct species with a polarity-specific rule table and proposes the accurate compound mass; getPeaklist returns the annotated peak table. CAMERA deliberately does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion; instead the rule table encodes known adduct/neutral-loss rules, and cross-polarity rules (only common, trusted combinations, since they can override single-polarity annotations) are applied when spectra measured in both ion modes are available. Annotations were validated by comparing proposed masses against a manually defined reference (39-compound MM39 mixture), where CAMERA correctly extracted masses for 89.7% and 90.3% of annotatable compounds in positive and negative modes.

## Related tools

- **CAMERA** (R/Bioconductor package that implements findIsotopes, groupFWHM, groupCorr and findAdducts to group and annotate isotope/adduct peaks and propose accurate masses) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream preprocessing framework that produces the peak list fed into xsAnnotate for annotation)

## Examples

```
xs <- xsAnnotate(xs=xcmsObj); xs <- groupFWHM(xs, ppm=10, mzabs=0.015); xs <- findIsotopes(xs); xs <- groupCorr(xs); xs <- findAdducts(xs, polarity='positive'); peaks <- getPeaklist(xs)
```

## Evaluation signals

- Proposed molecular mass matches the manually defined MM39 reference for ≥ 89.7% of annotatable compounds in positive mode (and ≥ 90.3% in negative mode)
- Each isotope annotation is consistent with m/z differences corresponding to a stable isotope of a constituent element
- Compound spectra contain no duplicated features (each feature assigned to exactly one pseudo-spectrum after grouping)
- Annotation rate improves when combining spectral information from both ionization modes compared with single-polarity analysis
- No ad-hoc assumption (e.g. most intense feature = [M+H]+) is used to select the quasi-molecular ion

## Limitations

- Missing mass annotations occurred when compound spectra contained too few explained features, or when several hypotheses had the same precedence scores, or when spectra contained only molecular and fragment ions with no further adducts
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- Cross-polarity rules can override single-polarity annotations, so they must be restricted to common and trusted combinations
- Accurate-mass extraction rate depends on the quality of upstream feature detection (XCMS) and on avoiding MS detector saturation, which shifts m/z and retention time centroids and truncates peak profiles

## Evidence

- [other] task card detailing the MM39 recovery workflow: [evidence span withheld — anchor status: not_found]
