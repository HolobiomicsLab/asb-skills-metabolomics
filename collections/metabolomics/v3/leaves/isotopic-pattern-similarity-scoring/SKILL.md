---
name: isotopic-pattern-similarity-scoring
description: Use when annotating ion peaks from an imaging MS feature matrix (e.g.,
  14,260 pixels x 3,044 ions) against a metabolite library and you need isotopic-pattern
  similarity as one of the multi-evidence scores for candidate ranking.
when_to_use_negative:
- Input is not a spatial (imaging MS) feature matrix with pixel coordinates and ion
  intensities — spatial correlation validation of isotope/adduct pairs is unavailable
- Library entries lack molecular formulas, so no theoretical isotopic pattern can
  be computed with enviPat isopattern
- You need single-candidate annotation only and do not want the multi-evidence multiplicity
  of retained candidates (374 single / 148 double / 147 multiple in the reference
  run)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SMAnalyst
  role: Web/R platform whose annotation pipeline performs the isotope/adduct identification
    and multi-evidence scoring
  repo: mzlab-research/SManalyst
- name: enviPat
  role: computes theoretical isotopic patterns (isopattern) and removes impossible
    adduct assignments (check_ded)
- name: msentropy
  role: quantifies similarity between observed and theoretical isotopic peak distributions
    via msentropy_similarity
- name: MetaboCoreUtils
  role: identifies isotopic peak pairs (isotopologues) and converts m/z to neutral
    mass over the adduct list (mz2mass)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:1ff1bd8b289f125f50e8a3b4a298df0c251bd194c33a5b5d80dd2440f6191b50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - SMAnalyst
  - enviPat
  - msentropy
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# isotopic-pattern-similarity-scoring

## Summary

Scores metabolite annotation candidates in spatial metabolomics by comparing an observed isotopic peak distribution to the theoretical isotopic pattern of a matched formula, computed with enviPat's isopattern and quantified with msentropy_similarity, alongside mass accuracy and adduct evidence.

## When to use

Use when annotating ion peaks from an imaging MS feature matrix (e.g., 14,260 pixels x 3,044 ions) against a metabolite library and you need isotopic-pattern similarity as one of the multi-evidence scores for candidate ranking.

## When NOT to use

- Input is not a spatial (imaging MS) feature matrix with pixel coordinates and ion intensities — spatial correlation validation of isotope/adduct pairs is unavailable
- Library entries lack molecular formulas, so no theoretical isotopic pattern can be computed with enviPat isopattern
- You need single-candidate annotation only and do not want the multi-evidence multiplicity of retained candidates (374 single / 148 double / 147 multiple in the reference run)

## Inputs

- Feature matrix CSV (columns 1-2: pixel X,Y coordinates; remaining columns: ion intensities at each m/z; rows = spatial pixels)
- Custom metabolite library with molecular formulas and adduct information (e.g., self-built mouse brain LC-MS/MS library, 1269 metabolites, identification levels 1-3)
- User-specified ppm mass-error tolerance and adduct list

## Outputs

- Per-ion annotation table written to annotation_results.csv (all retained candidate matches per ion with mass-accuracy, isotopic-similarity, and adduct-evidence scores)
- Summary of the number/proportion of identified ions and annotation multiplicity (single/double/multiple annotations)

## How to apply

First, within the user-specified ppm mass-error tolerance, detect isotopic peak pairs with MetaboCoreUtils::isotopologues and adduct pairs ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+), confirming each pair by spatial correlation with the default threshold of 0.5. Match neutral masses (MetaboCoreUtils::mz2mass over the adduct list) against the custom LC-MS/MS library, then prune impossible assignments with enviPat::check_ded (e.g., a molecule lacking oxygen cannot carry [M+H-H2O]+). For matched formulas with isotopic peaks, compute the theoretical isotopic pattern with enviPat::isopattern and quantify similarity to the observed pattern with msentropy::msentropy_similarity. Retain all candidate matches per ion and score each result by parent-ion mass matching accuracy, isotopic peak distribution similarity, and presence of a clear adduct form, rather than collapsing to one annotation per ion.

## Related tools

- **SMAnalyst** (Web/R platform whose annotation pipeline performs the isotope/adduct identification and multi-evidence scoring) — mzlab-research/SManalyst
- **enviPat** (computes theoretical isotopic patterns (isopattern) and removes impossible adduct assignments (check_ded))
- **msentropy** (quantifies similarity between observed and theoretical isotopic peak distributions via msentropy_similarity)
- **MetaboCoreUtils** (identifies isotopic peak pairs (isotopologues) and converts m/z to neutral mass over the adduct list (mz2mass))

## Examples

```
pattern <- isopattern(formula, isotopes, charge=1); score <- msentropy_similarity(observed_spectrum, pattern[[1]])
```

## Evaluation signals

- Identified-ion count and annotation multiplicity reproduce the reference: 669 identified ions with 374 single, 148 double, and 147 multiple annotations on the 14,260-pixel mouse brain dataset
- Secondary concordance check: 91% overlap of single annotations with MSIannotator
- Every isotopic/adduct peak pair used passes the spatial-correlation threshold (default 0.5)
- No retained candidate carries an adduct forbidden by its elemental composition (enviPat check_ded removes e.g. [M+H-H2O]+ for oxygen-free molecules)
- annotation_results.csv contains per-ion rows with all retained candidate matches and their mass-accuracy, isotopic-similarity, and adduct-evidence scores

## Limitations

- Requires R (>= v4.1.0) and the enviPat, msentropy, and MetaboCoreUtils packages installed alongside the SMAnalyst Shiny stack
- Mass-error tolerance and adduct list are user-specified, so counts and scores are sensitive to these settings; ion pairs must additionally pass spatial correlation (default 0.5) before scoring
- Non-monoisotopic peaks must first be removed; ions without a clear adduct form get neutral masses computed sequentially over the adduct list, which can leave ions unannotated
- Benchmarked on large-scale data (e.g., >14,000 pixels, >3000 ion peaks) with a mouse brain coronal section (AFAD-ESI (+), 100um resolution); much smaller or different-mode datasets are untested in the source evidence

## Evidence

- [readme] multi-evidence scoring: "Isotope/adduct identification with spatial correlation validation"
- [readme] scoring system: "Multi-evidence scoring (mass accuracy, isotope pattern, adduct form)"
- [abstract] annotation scoring system: "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
- [methods] isopattern + msentropy_similarity: [evidence span withheld — anchor status: not_found]
- [methods] candidate scoring: [evidence span withheld — anchor status: not_found]
- [methods] validation target: [evidence span withheld — anchor status: not_found]
