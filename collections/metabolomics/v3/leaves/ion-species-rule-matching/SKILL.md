---
name: ion-species-rule-matching
description: Use when LC/MS peak lists (e.g. XCMS feature tables) contain co-eluting
  features whose m/z differences match known adducts or losses and you need to reduce
  feature redundancy and identify quasi-molecular ions.
when_to_use_negative:
- When the input is an already-annotated or targeted feature table where every feature's
  ion species is known - the rule-matching step would re-derive what is already present.
- When the goal is a first-pass isotope grouping step rather than ion-species annotation;
  use CAMERA's findIsotopes/findIsotopesWithValidation or grouping algorithms (calcCiS,
  calcCaS, calcPC.lpc, calcPC.hcs) instead.
- When the compound spectrum contains only the molecular ion and fragment ions with
  no further adducts and the neutral loss is not in the rule set - direct annotation
  is not possible in that case.
- When data are saturated (detector over-range causes m/z and RT centroid shifts),
  because matching rules to shifted m/z differences would produce incorrect annotations.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: CAMERA
  role: R/Bioconductor package that supplies the dynamic rule generation and rule-matching
    functions (generateRules(), findAdducts()) used to annotate ion species within
    compound spectra.
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection/alignment framework producing the peak lists that
    CAMERA's rule matcher annotates.
- name: R
  role: Implementation environment; CAMERA is delivered as R packages from the Bioconductor
    repository for Windows, Mac OS and Linux.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:006ba897f1ac9a52ce281ed282273bd6af0fe99a52b1c82ce320a1a0762cfb88
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ion-species-rule-matching

## Summary

Dynamic rule-based matching of within-spectrum m/z differences to ion species (adducts, neutral losses, cluster ions), so LC/MS features from the same analyte are annotated with a single ion species and quasi-molecular ion instead of being treated as unrelated features.

## When to use

Use when LC/MS peak lists (e.g. XCMS feature tables) contain co-eluting features whose m/z differences match known adducts or losses and you need to reduce feature redundancy and identify quasi-molecular ions.

## When NOT to use

- When the input is an already-annotated or targeted feature table where every feature's ion species is known - the rule-matching step would re-derive what is already present.
- When the goal is a first-pass isotope grouping step rather than ion-species annotation; use CAMERA's findIsotopes/findIsotopesWithValidation or grouping algorithms (calcCiS, calcCaS, calcPC.lpc, calcPC.hcs) instead.
- When the compound spectrum contains only the molecular ion and fragment ions with no further adducts and the neutral loss is not in the rule set - direct annotation is not possible in that case.
- When data are saturated (detector over-range causes m/z and RT centroid shifts), because matching rules to shifted m/z differences would produce incorrect annotations.

## Inputs

- xsAnnotate object with grouped LC/MS peak data (e.g. from XCMS peak detection/alignment)
- Ion species list or rule table (adducts, neutral losses, cluster ions) for generateRules()
- Peak m/z differences within compound spectra
- ppm and mzabs tolerance parameters
- Optional database for findIsotopesWithValidation(object, maxcharge, ppm, mzabs, intval, validateIsotopePatterns, database)

## Outputs

- Annotated peak table with ion species assignment per feature
- Quasi-molecular ion annotation (proposed accurate compound mass) for each compound spectrum
- Annotation-completeness comparison against CAMERA's reported quasi-molecular ion annotations
- Group/conflict-resolved annotation hypotheses

## How to apply

Load the grouped peak data as an xsAnnotate object from CAMERA (Bioconductor, R). Build a dynamic rule set with generateRules() from the ion species lists; each rule encodes the ion species' mass difference to the molecular mass, its ion charge, and the number of molecules it contains. Then call findAdducts() to match within-spectrum m/z differences to these rules using ppm and mzabs tolerances, merging same-mass hypotheses and resolving conflicts between annotation groups. CAMERA deliberately avoids the ad-hoc heuristic of assuming the most intense feature is the [M+H]+-ion; the rule table, not peak intensity, decides the annotation. Cross-polarity rules should contain only common and trusted combinations, because they can override single-polarity annotations. Finally evaluate completeness by comparing against CAMERA's reported quasi-molecular ion annotations and save the annotated peak table with an ion species assignment for each feature.

## Related tools

- **CAMERA** (R/Bioconductor package that supplies the dynamic rule generation and rule-matching functions (generateRules(), findAdducts()) used to annotate ion species within compound spectra.) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection/alignment framework producing the peak lists that CAMERA's rule matcher annotates.)
- **R** (Implementation environment; CAMERA is delivered as R packages from the Bioconductor repository for Windows, Mac OS and Linux.)

## Examples

```
findAdducts(xs, ppm=5, mzabs=0.001)
```

## Evaluation signals

- Each annotated feature's m/z difference to the proposed quasi-molecular ion is consistent with one rule's mass difference, charge, and molecule count within the ppm/mzabs tolerance used.
- Annotation completeness, measured by comparing CAMERA's annotations against the known quasi-molecular ions, should match the paper's reported extraction accuracy (89.7% positive, 90.3% negative ion mode on the MM39 validation mixture).
- No conflicting ion species assignments remain between annotation groups after the merge/conflict-resolution step.
- Cross-polarity (both ion mode) annotations are only accepted when the cross-polarity rule table entry is a common and trusted combination.
- The output peak table carries an explicit ion species label for every feature, not just a cluster grouping.

## Limitations

- CAMERA does not use ad-hoc heuristics such as assuming the most intense feature in a spectrum is the [M+H]+-ion, so intensity-only reasoning will mislead the annotation.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly, unless the neutral loss is added to the rule set.
- Missing molecular mass annotations occur when the compound spectrum has too few explained features, or when several hypotheses tie on precedence scores.
- MS detector saturation causes m/z and retention time centroid shifts and truncated peak profiles, which corrupt rule matching.
- Cross-polarity rules must be limited to common and trusted combinations, because they can override single-polarity annotations.

## Evidence

- [readme] Implementation and availability: [evidence span withheld — anchor status: not_found]
