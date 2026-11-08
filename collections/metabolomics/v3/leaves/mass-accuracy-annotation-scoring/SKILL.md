---
name: mass-accuracy-annotation-scoring
description: Use when a spatial metabolomics feature matrix (pixels × m/z ion peaks)
  must be matched against a custom or built-in metabolite database to obtain per-ion
  annotations with multiplicity counts and annotation scores.
when_to_use_negative:
- Input is raw MS imaging data rather than a pre-extracted feature matrix — SMAnalyst
  expects a pixel × m/z intensity table, not vendor raw files
- The goal is differential expression or spatial clustering (metabolite/pixel clustering,
  ROI comparison) rather than metabolite identification
- No reference metabolite database or library is available and only de novo peak picking/marker-ion
  discovery (e.g. SmartGate-style graph-attention peak picking) is desired
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SMAnalyst
  role: Web platform hosting the metabolite annotation pipeline with multi-evidence
    scoring (mass accuracy, isotope pattern, adduct form)
  repo: https://github.com/mzlab-research/SManalyst
- name: MetaboCoreUtils
  role: R package used for isotopologue peak-pair detection and mz2mass neutral-mass
    conversion over the adduct list
- name: enviPat
  role: check_ded filters impossible adduct assignments; isopattern computes theoretical
    isotopic patterns for matched formulas
- name: msentropy
  role: msentropy_similarity quantifies similarity between observed and theoretical
    isotopic peak patterns
- name: R (>= v4.1.0)
  role: Runtime environment for the local SMAnalyst installation and annotation pipeline
  repo: https://github.com/mzlab-research/SManalyst
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:f750d544a9b8031e5f2d7cdfdd31fe0d794dd6d41ae1608c676c3df391e3098c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - MetaboCoreUtils
  - R (>= v4.1.0)
  - SMAnalyst
  - enviPat
  - msentropy
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-accuracy-annotation-scoring

## Summary

Annotate spatial metabolomics ion peaks by matching neutral masses against an LC-MS/MS reference library and scoring each candidate on mass accuracy, isotopic-pattern similarity, and adduct evidence, as implemented in SMAnalyst's R annotation pipeline.

## When to use

Use when a spatial metabolomics feature matrix (pixels × m/z ion peaks) must be matched against a custom or built-in metabolite database to obtain per-ion annotations with multiplicity counts and annotation scores.

## When NOT to use

- Input is raw MS imaging data rather than a pre-extracted feature matrix — SMAnalyst expects a pixel × m/z intensity table, not vendor raw files
- The goal is differential expression or spatial clustering (metabolite/pixel clustering, ROI comparison) rather than metabolite identification
- No reference metabolite database or library is available and only de novo peak picking/marker-ion discovery (e.g. SmartGate-style graph-attention peak picking) is desired

## Inputs

- Feature matrix CSV (columns 1–2 = pixel X,Y coordinates; remaining columns = ion intensities at each m/z value; e.g. 14,260 pixels × 3,044 ions from a mouse brain coronal section, AFAD-ESI (+), 100 μm resolution)
- Metabolite reference library (custom upload or built-in; e.g. self-built mouse brain LC-MS/MS library, 1269 metabolites, identification levels 1–3; HMDB/KEGG/LIPIDMAPS also supported)
- User-specified ppm mass-error tolerance
- User-specified adduct list (e.g. [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+)

## Outputs

- Per-ion annotation table written to annotation_results.csv
- Counts and proportions of identified ions (e.g. 669 of 3044 ions identified)
- Annotation multiplicity breakdown (single / double / ≥3 annotations per ion)
- Per-ion annotation scores (mass accuracy, isotopic similarity, adduct evidence)

## How to apply

Load the feature matrix and the metabolite library (e.g. a self-built mouse brain LC-MS/MS library of 1269 metabolites, identification levels 1–3) into SMAnalyst's R (>= 4.1.0) annotation pipeline. First identify isotopic peak pairs with the MetaboCoreUtils isotopologues function within the user-specified ppm mass-error tolerance, and adduct ion pairs for [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, and [M+H-H2O]+, confirming each pair by spatial correlation with the default threshold of 0.5; then remove non-monoisotopic peaks. For ions with a clear adduct form, match the calculated neutral mass directly against the library; for ions without one, compute neutral masses sequentially over the user-specified adduct list using the MetaboCoreUtils mz2mass function, and eliminate impossible adduct assignments with the enviPat check_ded function (e.g. a molecule lacking oxygen cannot carry an [M+H-H2O]+ adduct). For matched formulas with isotopic peaks, compute the theoretical isotopic pattern with enviPat isopattern and quantify similarity to the observed pattern with msentropy msentropy_similarity. Retain all candidate matches per ion and score each result by parent-ion mass matching accuracy, isotopic peak distribution similarity, and presence of a clear adduct form before summarizing single/double/multiple annotation multiplicities.

## Related tools

- **SMAnalyst** (Web platform hosting the metabolite annotation pipeline with multi-evidence scoring (mass accuracy, isotope pattern, adduct form)) — https://github.com/mzlab-research/SManalyst
- **MetaboCoreUtils** (R package used for isotopologue peak-pair detection and mz2mass neutral-mass conversion over the adduct list)
- **enviPat** (check_ded filters impossible adduct assignments; isopattern computes theoretical isotopic patterns for matched formulas)
- **msentropy** (msentropy_similarity quantifies similarity between observed and theoretical isotopic peak patterns)
- **R (>= v4.1.0)** (Runtime environment for the local SMAnalyst installation and annotation pipeline) — https://github.com/mzlab-research/SManalyst

## Examples

```
Rscript -e "shiny::runApp('SManalyst')"  # then Upload tab -> feature matrix CSV -> Metabolite Annotation tab -> select 1269-metabolite library, set ppm tolerance, review annotation_results.csv
```

## Evaluation signals

- Reproduce the reported annotation result: 669 identified ions out of 3044 unique ion peaks, with 374 single, 148 double, and 147 multiple (≥3) annotations against the 1269-metabolite mouse brain library
- Secondary concordance check: ~91% overlap of single annotations with MSIannotator
- Every isotope/adduct peak pair passes the spatial correlation threshold of 0.5 before neutral-mass matching
- Impossible adduct assignments are removed (e.g. check_ded rejects [M+H-H2O]+ for molecules lacking oxygen)
- annotation_results.csv exists and contains one row per ion with retained candidate matches and composite annotation scores

## Limitations

- Annotation is library-dependent: only ions matching entries in the uploaded/built-in database (custom, HMDB/KEGG/LIPIDMAPS) can be identified; a 1269-metabolite library annotated 669 of 3044 ions (~23%)
- Mass-error tolerance and adduct list are user-specified and directly affect candidate multiplicity (148 ions had two annotations, 147 had ≥3), so overly permissive tolerances inflate multiple annotations
- Large-scale inputs (e.g. >14,000 pixels, >3000 ion peaks) are supported but were only validated on a mouse brain AFAD-ESI (+) 100 μm dataset

## Evidence

- [intro] multi-evidence annotation scoring: "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
- [readme] isotope/adduct identification: "Isotope/adduct identification with spatial correlation validation
    *   Multi-evidence scoring (mass accuracy, isotope pattern, adduct form)"
- [readme] annotation workflow tab: "**Metabolite Annotation**:
    *   Select database (custom upload or built-in)
    *   Set mass error tolerance (ppm)
    *   Review annotation scores"
- [readme] test dataset scale: "Test dataset: **Mouse brain coronal section**
*   Acquisition: AFAD-ESI (+) mode, 100μm resolution
*   Content: 14,260 pixels × 3,044 ions"
- [abstract] validated scale claim: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
