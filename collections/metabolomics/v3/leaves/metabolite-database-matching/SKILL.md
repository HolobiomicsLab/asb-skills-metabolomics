---
name: metabolite-database-matching
description: Use when ion peaks from a spatial metabolomics feature matrix must be
  matched against a reference LC-MS/MS library (e.g. a 1269-metabolite mouse brain
  library) and each annotation must be scored by mass accuracy, isotope pattern, and
  adduct evidence.
when_to_use_negative:
- The input is raw mass-spectrometry data rather than a pre-processed feature matrix
  with pixel coordinates and m/z intensity columns, since SMAnalyst's annotation step
  requires the feature-matrix CSV format
- The goal is differential metabolite detection or spatial pixel/metabolite clustering
  rather than identifying which metabolites the ion peaks correspond to
- No reference LC-MS/MS library (custom or built-in HMDB/KEGG/LIPIDMAPS) is available,
  since this skill matches against a curated database rather than performing de novo
  assignment
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMAnalyst
  role: Integrated web/R platform whose metabolite annotation module performs the
    database matching with multi-evidence scoring
  repo: https://github.com/mzlab-research/SManalyst
- name: MetaboCoreUtils
  role: R package used for isotopologues peak-pair identification and mz2mass neutral-mass
    conversion across the adduct list
- name: enviPat
  role: R package used for check_ded elimination of impossible adduct assignments
    and isopattern computation of theoretical isotopic patterns
- name: msentropy
  role: Provides the msentropy_similarity method to quantify similarity between observed
    and theoretical isotopic peak distributions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:cd67a3b06552414a1b2efced744e21335474d12a2fc8c7c9c1613ae8b4d977ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - SMAnalyst
  - enviPat
  - msentropy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolite-database-matching

## Summary

Annotates ion peaks in a spatial metabolomics feature matrix against an LC-MS/MS metabolite library using multi-evidence scoring that combines parent-ion mass accuracy, isotopic-pattern similarity, and adduct-form evidence. Implemented in SMAnalyst's R pipeline with MetaboCoreUtils, enviPat, and msentropy.

## When to use

Use when ion peaks from a spatial metabolomics feature matrix must be matched against a reference LC-MS/MS library (e.g. a 1269-metabolite mouse brain library) and each annotation must be scored by mass accuracy, isotope pattern, and adduct evidence.

## When NOT to use

- The input is raw mass-spectrometry data rather than a pre-processed feature matrix with pixel coordinates and m/z intensity columns, since SMAnalyst's annotation step requires the feature-matrix CSV format
- The goal is differential metabolite detection or spatial pixel/metabolite clustering rather than identifying which metabolites the ion peaks correspond to
- No reference LC-MS/MS library (custom or built-in HMDB/KEGG/LIPIDMAPS) is available, since this skill matches against a curated database rather than performing de novo assignment

## Inputs

- Feature matrix CSV: columns 1-2 are pixel coordinates (X, Y), subsequent columns are ion intensities at each m/z value, one row per spatial pixel (e.g. 14,260 pixels x 3,044 ions from a mouse brain coronal section, AFADESI-ESI (+), 100 um resolution)
- Reference LC-MS/MS metabolite library (custom upload or built-in; e.g. 1269 mouse brain metabolites at identification levels 1-3) with molecular formulas
- User-specified ppm mass-error tolerance and adduct list

## Outputs

- Per-ion annotation table written to annotation_results.csv with all retained candidate matches and their multi-evidence scores
- Summary of the number/proportion of identified ions and the multiplicity distribution of annotations (single, double, multiple)

## How to apply

Load the feature matrix (e.g. 14,260 spatial pixels x 3044 unique ion peaks) and the LC-MS/MS library (e.g. 1269 metabolites at identification levels 1-3) into SMAnalyst's R (>= 4.1.0) annotation pipeline. First identify isotopic peak pairs with the MetaboCoreUtils isotopologues function within the user-specified ppm mass-error tolerance, and detect adduct ion pairs for common forms ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+), confirming each pair by spatial correlation with the default threshold of 0.5. Remove non-monoisotopic peaks, then match neutral masses against the library: for ions with a clear adduct form match the calculated neutral mass directly, and for ions without one compute neutral masses sequentially over the user-specified adduct list with the MetaboCoreUtils mz2mass function. Eliminate chemically impossible adduct assignments with the enviPat check_ded function (e.g. a molecule lacking oxygen cannot carry an [M+H-H2O]+ adduct), and for matched formulas with isotopic peaks compute the theoretical isotopic pattern with enviPat isopattern and quantify similarity to the observed pattern using the msentropy msentropy_similarity method. Retain all candidate matches per ion and score each result comprehensively by parent-ion mass matching accuracy, isotopic peak distribution similarity, and presence of a clear adduct form, then summarize the number/proportion of identified ions and annotation multiplicity (single/double/multiple) and write the per-ion annotation table to annotation_results.csv.

## Related tools

- **SMAnalyst** (Integrated web/R platform whose metabolite annotation module performs the database matching with multi-evidence scoring) — https://github.com/mzlab-research/SManalyst
- **MetaboCoreUtils** (R package used for isotopologues peak-pair identification and mz2mass neutral-mass conversion across the adduct list)
- **enviPat** (R package used for check_ded elimination of impossible adduct assignments and isopattern computation of theoretical isotopic patterns)
- **msentropy** (Provides the msentropy_similarity method to quantify similarity between observed and theoretical isotopic peak distributions)

## Examples

```
neutral_masses <- MetaboCoreUtils::mz2mass(mz_values, adducts = c("[M+H]+","[M+K]+","[M+Na]+","[M+NH4]+","[M+H-H2O]+"); enviPat::check_ded(adduct_table); msentropy::msentropy_similarity(observed_pattern, enviPat::isopattern(formula))
```

## Evaluation signals

- On the reference Dataset 1 mouse brain run, the pipeline must identify 669 ions, of which 374 had a single annotation, 148 had two annotations, and 147 had three or more annotations
- Secondary concordance check: 91% overlap of single annotations with MSIannotator
- Every confirmed isotope/adduct peak pair must pass the spatial-correlation threshold (default 0.5)
- check_ded must flag chemically impossible adducts, e.g. no molecule lacking oxygen is assigned [M+H-H2O]+
- annotation_results.csv is produced containing per-ion candidate matches with scores by mass accuracy, isotopic similarity, and adduct evidence

## Limitations

- Annotation multiplicity is high: 148 ions received two annotations and 147 received three or more, so downstream use requires disambiguation via the composite score
- The default adduct scope is limited to common forms ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+); rarer adducts need an explicit user-specified adduct list
- Requires R >= 4.1.0 and the full CRAN/Bioconductor dependency set (enviPat, msentropy, MetaboCoreUtils, etc.) for local installation
- Annotation quality depends on the library: the reference run used a self-built 1269-metabolite mouse brain LC-MS/MS library at identification levels 1-3

## Evidence

- [readme] multi-evidence scoring: "Isotope/adduct identification with spatial correlation validation"
- [readme] multi-evidence scoring 2: "Multi-evidence scoring (mass accuracy, isotope pattern, adduct form)"
- [readme] databases: "Support for custom databases and public repositories (HMDB/KEGG/LIPIDMAPS)"
- [intro] annotation module: "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
