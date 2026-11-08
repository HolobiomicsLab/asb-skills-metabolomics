---
name: compound-mass-reference-matching
description: Use when a CAMERA-annotated LC/MS feature list must be compared to a
  reference compound list (e.g. MM39) to count correctly extracted molecular masses
  per ion mode.
when_to_use_negative:
- When the input is already a curated, manually annotated compound table rather than
  a raw XCMS peak list from untargeted LC/MS data
- When only one ionization mode was acquired and the cross-polarity verification path
  cannot be exercised
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CAMERA
  role: Integrates feature grouping and isotope/adduct annotation, proposes accurate
    compound masses
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Generates the peak lists of detected features that CAMERA groups and annotates
- name: R
  role: Implementation language; CAMERA is installed from Bioconductor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:46460c812dc684c42b4cf0eacb9ca12a8a1f02c02d942bbe65a17d10ebb87fee
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# compound-mass-reference-matching

## Summary

Proposes accurate compound masses from CAMERA-annotated isotope/adduct peak groups and cross-references them against a curated reference list (sum formulas, molar masses, PubChem IDs) to quantify how well annotation recovers true analyte masses in a complex matrix.

## When to use

Use when a CAMERA-annotated LC/MS feature list must be compared to a reference compound list (e.g. MM39) to count correctly extracted molecular masses per ion mode.

## When NOT to use

- When the input is already a curated, manually annotated compound table rather than a raw XCMS peak list from untargeted LC/MS data
- When only one ionization mode was acquired and the cross-polarity verification path cannot be exercised

## Inputs

- XCMS peak list from LC/MS runs (both ion modes)
- Reference compound list with sum formulas, molar masses, PubChem IDs (e.g. MM39 spiked at 20, 5, 1 and 0.2 µM)
- Polarity rule set for findAdducts (positive and negative ion modes)

## Outputs

- Annotated peak table from getPeaklist with proposed compound masses
- Per-ion-mode counts of correctly extracted molecular masses (e.g. 89.7% positive, 90.3% negative)

## How to apply

Build an xsAnnotate object from an XCMS peak list, run groupFWHM for retention-time co-elution grouping, findIsotopes for isotope peaks, groupCorr using EIC intensity correlation across the spiked samples, and findAdducts with the polarity-appropriate rule set to label adduct species and propose the accurate compound mass for each pseudo-spectrum. Extract the annotated peak list with getPeaklist and compare the derived masses against the manually defined reference annotations (e.g. the MM39 mixture of 39 known compounds spanning 161–822 Da, with sum formulas, molar masses and PubChem IDs), counting correctly extracted masses per positive and negative ion mode. Reference the MM39 recovery datasets (pure solution plus extract spiked at 20, 5, 1 and 0.2 µM). Exclude compounds with retention time below 45 s before comparison, since those are outside the analysis window. Rationale: CAMERA avoids ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion; instead the mass proposal derives from the annotated pseudo-spectrum.

## Related tools

- **CAMERA** (Integrates feature grouping and isotope/adduct annotation, proposes accurate compound masses) — https://github.com/sneumann/CAMERA
- **XCMS** (Generates the peak lists of detected features that CAMERA groups and annotates)
- **R** (Implementation language; CAMERA is installed from Bioconductor)

## Examples

```
R: xs <- xsAnnotate(peaks); xs <- groupFWHM(xs); xs <- findIsotopes(xs); xs <- groupCorr(xs); xs <- findAdducts(xs, polarity='positive'); pl <- getPeaklist(xs)
```

## Evaluation signals

- Correctly extracted molecular masses per ion mode should be ≈89.7% (positive) and ≈90.3% (negative) against the MM39 reference annotations
- Every pseudo-spectrum must receive a proposed accurate mass only when it contains a sufficient number of explained features; missing annotations should be traceable to too few explained features, tied precedence scores, or spectra with only molecular and fragment ions
- Cross-polarity rules may override single-polarity annotations, so verify that annotation verification using the opposite ion mode does not contradict the final mass assignment
- Feature count in the reference-matched table should be reduced relative to the raw feature list (prior work reports reductions down to 50%)
- Compounds eluting before 45 s should be absent from the comparison results

## Limitations

- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- Missing annotations occur when the compound spectrum contains too few explained features or when several hypotheses share the same precedence score
- Sample concentration must avoid MS detector saturation, which shifts m/z and retention time centroids and truncates peak profiles
- Cross-polarity rules should only contain common and trusted combinations because they can override single-polarity annotations

## Evidence
