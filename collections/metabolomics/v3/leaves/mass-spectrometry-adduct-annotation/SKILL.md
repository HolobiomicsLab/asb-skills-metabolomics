---
name: mass-spectrometry-adduct-annotation
description: Use when an LC/MS feature table contains multiple features per analyte
  (isotope/adduct/fragment series) and you must assign quasi-molecular ion species,
  charge states, or neutral losses to reduce feature-list redundancy before statistical
  analysis or compound identification.
when_to_use_negative:
- The input is already an annotated, de-isotoped feature list with one feature per
  analyte — annotation would add nothing and risk misassignment.
- The compound spectrum contains only the molecular ion and fragment ions with no
  further adducts and the neutral loss has not been added to the rule set — CAMERA
  cannot annotate such cases directly.
- The data come from a single ungrouped peak list without prior isotope/retention-time/intensity-correlation
  grouping; rule matching operates within compound spectra, not raw peak lists.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Bioconductor R package implementing compound-spectra extraction, grouping,
    dynamic rule generation, and adduct/neutral-loss/cluster-ion annotation
  repo: https://travis-ci.org/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection and alignment producing the peak table fed into
    CAMERA
- name: R
  role: Runtime environment; CAMERA packages for Windows, Mac OS and Linux are available
    from the Bioconductor repository
  repo: https://travis-ci.org/sneumann/CAMERA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:45c971339e92e4a5ef7abf3f514e8fc3312179c2e2c05aaf21ff28d700db72af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mass-spectrometry-adduct-annotation

## Summary

Annotate the ion species (adducts, neutral losses, cluster ions, isotopes) underlying each LC/MS feature by matching within-compound-spectrum m/z differences against a dynamic rule set, as implemented in the CAMERA Bioconductor package for untargeted metabolomics.

## When to use

Use when an LC/MS feature table contains multiple features per analyte (isotope/adduct/fragment series) and you must assign quasi-molecular ion species, charge states, or neutral losses to reduce feature-list redundancy before statistical analysis or compound identification.

## When NOT to use

- The input is already an annotated, de-isotoped feature list with one feature per analyte — annotation would add nothing and risk misassignment.
- The compound spectrum contains only the molecular ion and fragment ions with no further adducts and the neutral loss has not been added to the rule set — CAMERA cannot annotate such cases directly.
- The data come from a single ungrouped peak list without prior isotope/retention-time/intensity-correlation grouping; rule matching operates within compound spectra, not raw peak lists.

## Inputs

- Grouped peak table / xsAnnotate object from CAMERA with peak groups spanning isotope, adduct, and fragment features
- LC/MS centroided peak lists from UPLC/ESI-QTOF data (e.g. Bruker MicrOTOF-Q, m/z 100–1000, 3 Hz), acquired in positive and/or negative ion mode
- Rule configuration: ion species lists consumed by generateRules() (adducts, neutral losses, cluster ions, charges)
- Optional polarity-specific rule tables and a cross-polarity rule table containing only common, trusted combinations

## Outputs

- Annotated peak table with ion species assignment (e.g. [M+H]+, [M+Na]+, [M+NH4]+, [2M+H]+, neutral losses), charge, and quasi-molecular-ion annotation per feature
- Compound spectra grouping features by analyte with the proposed accurate compound mass
- Cross-polarity annotation verification results when both ion modes are available

## How to apply

Start from a grouped peak table (xsAnnotate object) produced by CAMERA's feature-grouping steps, then generate the dynamic rule set with generateRules(): each rule encodes an ion species' mass difference to the molecular mass, its charge, and its number of molecules, built from lists of observable ion species. Within each compound spectrum, findAdducts() matches observed m/z differences between features against these rules using ppm (e.g. 5 ppm on QTOF data) and mzabs absolute-mass tolerances, then merges same-mass hypotheses and resolves conflicts between annotation groups. For data acquired in both polarities, use CAMERA's novel annotation-verification algorithm that combines spectral information across opposite ion modes; keep the cross-polarity rule table restricted to common and trusted combinations, since these rules can override single-polarity annotations. Importantly, CAMERA does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion — annotations come solely from the rule set. On the MM39 evaluation mixture (39 standards, m/z 161–822), this correctly assigned ion species for 89.7% (positive) and 90.3% (negative) of annotatable compounds; compounds whose spectra contain only a molecular ion and fragments remain unannotated unless the neutral loss is added to the rule set.

## Related tools

- **CAMERA** (Bioconductor R package implementing compound-spectra extraction, grouping, dynamic rule generation, and adduct/neutral-loss/cluster-ion annotation) — https://travis-ci.org/sneumann/CAMERA
- **XCMS** (Upstream feature detection and alignment producing the peak table fed into CAMERA)
- **R** (Runtime environment; CAMERA packages for Windows, Mac OS and Linux are available from the Bioconductor repository) — https://travis-ci.org/sneumann/CAMERA

## Examples

```
annot <- xsAnnotate(xs); annot <- groupFWHM(annot); rules <- generateRules(annot); annot <- findAdducts(annot, ppm=5, mzabs=0.01)
```

## Evaluation signals

- Percentage of annotatable compounds in a standard mixture (e.g. MM39) with a correct ion species assignment should approach 89.7% in positive and 90.3% in negative ion mode.
- Every annotation must be traceable to a rule in the dynamic rule set — no ad-hoc assumption that the most intense feature is the [M+H]+-ion.
- For matched hypotheses with equal precedence scores, annotation should be flagged as ambiguous rather than forced; missing molecular-mass annotations often stem from too few explained features.
- Cross-polarity annotations should only override single-polarity annotations when derived from trusted, common combinations.
- Quasi-molecular ion proposed by CAMERA should match the accurate mass used for subsequent elemental composition calculation on known standards.

## Limitations

- Compound spectra with too few explained features, or several hypotheses tied on precedence scores, cannot be annotated and yield missing molecular-mass annotations.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Cross-polarity rules can override single-polarity annotations, so untrusted or rare combinations in the cross-polarity rule table can propagate errors.
- MS detector saturation from overly concentrated samples shifts m/z and retention-time centroids and truncates peak profiles, degrading feature grouping and annotation.
- CAMERA does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion, so absent supporting adducts the quasi-molecular ion may remain unassigned.

## Evidence

- [methods] CAMERA's dynamic rule set is built by combining lists of observable ion species, and each rule encodes a specific ion species' mass difference to the molecular mass, its ion charge, and the number of molecules it contains.: [evidence span withheld — anchor status: not_found]
