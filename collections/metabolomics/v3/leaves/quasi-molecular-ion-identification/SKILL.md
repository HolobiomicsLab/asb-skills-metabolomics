---
name: quasi-molecular-ion-identification
description: Decision support for annotating ion species (adducts, neutral losses,
  cluster ions) in grouped LC/MS compound spectra and revealing quasi-molecular ions
  to reduce feature-list redundancy.
when_to_use_negative:
- Peak data are not yet grouped into compound spectra (e.g. raw XCMS peak list before
  grouping) — run grouping first, since the rule matching operates within a spectrum.
- The analyte's ion species is absent from the rule set (e.g. an unusual cluster ion)
  and no neutral-loss can be added — annotation will fail or misassign.
- You only need a rough pseudo-molecular-ion guess and accept ad-hoc heuristics; CAMERA
  explicitly avoids assuming the most intense feature is the [M+H]+-ion, so quick
  heuristics belong elsewhere.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Bioconductor package providing generateRules() and findAdducts() to build
    the dynamic rule table and annotate adducts, neutral losses and cluster ions within
    compound spectra
  repo: https://github.com/sneumann/CAMERA
- name: R
  role: Implementation language in which CAMERA's annotation algorithms are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:a720459cfc737e079c39f8695916b48764c96750a14d2f4e1d23bcf389647eab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quasi-molecular-ion-identification

## Summary

Identify the quasi-molecular ion of each compound by building a dynamic adduct/neutral-loss/cluster-ion rule set from ion-species lists, scoring hypotheses from within-spectrum m/z differences, and revealing the molecular mass that enables elemental composition calculation and tandem-MS follow-up.

## When to use

Decision support for annotating ion species (adducts, neutral losses, cluster ions) in grouped LC/MS compound spectra and revealing quasi-molecular ions to reduce feature-list redundancy.

## When NOT to use

- Peak data are not yet grouped into compound spectra (e.g. raw XCMS peak list before grouping) — run grouping first, since the rule matching operates within a spectrum.
- The analyte's ion species is absent from the rule set (e.g. an unusual cluster ion) and no neutral-loss can be added — annotation will fail or misassign.
- You only need a rough pseudo-molecular-ion guess and accept ad-hoc heuristics; CAMERA explicitly avoids assuming the most intense feature is the [M+H]+-ion, so quick heuristics belong elsewhere.

## Inputs

- Grouped peak table / xsAnnotate object with retention time, m/z and intensity per feature
- Ion species list defining observable adducts, neutral losses and cluster ions per polarity
- ppm and mzabs mass tolerances
- Optional rule table for cross-polarity verification (common, trusted combinations only)

## Outputs

- Annotated peak table with ion species assignment and charge per feature
- Proposed accurate compound (molecular) mass per compound spectrum
- Set of unannotated features with the reason (too few explained features, tied precedence scores, or spectrum containing only molecular/fragment ions)

## How to apply

Construct the dynamic rule set with CAMERA's generateRules(): each rule encodes a specific ion species' mass difference to the molecular mass, its ion charge, and the number of molecules it contains. For each grouped spectrum, use findAdducts() to match within-spectrum m/z differences to these rules using the ppm tolerance and mzabs parameters. Merge hypotheses sharing the same mass and resolve conflicts between annotation groups, without ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion. When spectra contain only a molecular ion and fragment ions, add the corresponding neutral loss to the rule set before re-annotation. For polarity-switched acquisitions, apply cross-polarity verification restricted to common and trusted rule combinations, since these rules can override single-polarity annotations. Output is an annotated peak table with an ion-species assignment and proposed compound mass per feature.

## Related tools

- **CAMERA** (Bioconductor package providing generateRules() and findAdducts() to build the dynamic rule table and annotate adducts, neutral losses and cluster ions within compound spectra) — https://github.com/sneumann/CAMERA
- **R** (Implementation language in which CAMERA's annotation algorithms are executed)

## Examples

```
ann <- generateRules(object, ionlist, neutralloss, neutraladduction); object <- findAdducts(ann, ppm = 5, mzabs = 0.01)
```

## Evaluation signals

- Per-feature annotation includes ion species, charge and molecule count consistent with the matched rule
- Proposed quasi-molecular masses are reproducible when the same rule set is rebuilt via generateRules()
- Annotation of compounds in the MM39 standard mixture matches expected quasi-molecular ions; the paper reports accurate-mass extraction for 89.7% (positive) and 90.3% (negative) of annotatable compounds
- Cross-polarity annotations agree between positive and negative modes and do not contradict single-polarity annotations
- Features left unannotated carry a diagnosable cause (insufficient explained features, tied precedence, or molecular/fragment-only spectra)

## Limitations

- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- Missing annotations occur when the compound spectrum contains too few explained features or when several hypotheses have the same precedence scores
- Cross-polarity rules can override single-polarity annotations, so the cross-polarity rule table must contain only common and trusted combinations
- CAMERA does not use the ad-hoc heuristic of assuming the most intense feature is the [M+H]+-ion; users must not supply such shortcuts

## Evidence

- [methods] Dynamic rule set encodes mass difference, charge, molecule count: [evidence span withheld — anchor status: not_found]
