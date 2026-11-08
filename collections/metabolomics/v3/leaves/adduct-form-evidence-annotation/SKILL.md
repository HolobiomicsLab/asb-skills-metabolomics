---
name: adduct-form-evidence-annotation
description: Use when annotating m/z ion peaks from an imaging MS feature matrix against
  a metabolite library (e.g., a 1269-metabolite mouse brain LC-MS/MS library) and
  you must report how many ions receive single vs. multiple annotations.
when_to_use_negative:
- Input is raw mass-spectrometry data rather than a processed feature matrix in the
  required CSV format (X, Y columns followed by m/z intensity columns)
- No metabolite reference library or formula database is available, since the method
  matches neutral masses against a library and cannot do de novo identification
- The goal is spatial pattern discovery (metabolite/pixel clustering) or differential
  analysis rather than metabolite annotation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Integrated R/Shiny platform whose Metabolite Annotation module executes the
    adduct/isotope identification and multi-evidence scoring workflow
- name: MetaboCoreUtils
  role: Provides isotopologues (isotopic peak-pair detection within ppm tolerance)
    and mz2mass (m/z-to-neutral-mass conversion over the adduct list)
- name: enviPat
  role: check_ded removes chemically impossible adduct assignments; isopattern computes
    theoretical isotopic patterns for matched formulas
- name: msentropy
  role: msentropy_similarity quantifies similarity between observed and theoretical
    isotopic peak patterns
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:7184c788d2630642702dee65c9e5768cec1223c0717197a08e59a729ee838b2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboCoreUtils
  - SMAnalyst
  - enviPat
  - msentropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# adduct-form-evidence-annotation

## Summary

Annotates ion peaks in a spatial metabolomics feature matrix against a metabolite library using multi-evidence scoring: parent-ion mass accuracy, isotopic-pattern similarity, and presence of a clear adduct form, as implemented in SMAnalyst's R annotation pipeline.

## When to use

Use when annotating m/z ion peaks from an imaging MS feature matrix against a metabolite library (e.g., a 1269-metabolite mouse brain LC-MS/MS library) and you must report how many ions receive single vs. multiple annotations.

## When NOT to use

- Input is raw mass-spectrometry data rather than a processed feature matrix in the required CSV format (X, Y columns followed by m/z intensity columns)
- No metabolite reference library or formula database is available, since the method matches neutral masses against a library and cannot do de novo identification
- The goal is spatial pattern discovery (metabolite/pixel clustering) or differential analysis rather than metabolite annotation

## Inputs

- Feature matrix CSV (columns 1-2: pixel X/Y coordinates; subsequent columns: ion intensities at each m/z; one row per spatial pixel, e.g., 14,260 pixels x 3,044 ions)
- Custom metabolite library with molecular formulas and identification levels (e.g., mouse brain LC-MS/MS library, 1269 metabolites, levels 1-3)
- User-specified mass error tolerance (ppm) and adduct list

## Outputs

- Per-ion annotation table written to annotation_results.csv
- Summary counts of identified ions and annotation multiplicity (single / double / three-or-more annotations)

## How to apply

Load the feature matrix (e.g., 14,260 spatial pixels x 3044 unique ion peaks) and the metabolite library into SMAnalyst's R (>= 4.1.0) annotation pipeline. First identify isotopic peak pairs with MetaboCoreUtils::isotopologues within the user-specified ppm mass-error tolerance and adduct ion pairs for [M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, and [M+H-H2O]+, confirming each pair by spatial correlation with the default threshold of 0.5. Remove non-monoisotopic peaks, then match neutral masses against the library: use the calculated neutral mass directly when a clear adduct form exists, otherwise compute neutral masses sequentially over the adduct list with MetaboCoreUtils::mz2mass. Eliminate chemically impossible adduct assignments with enviPat::check_ded (e.g., a molecule lacking oxygen cannot carry [M+H-H2O]+). For matched formulas with isotopic peaks, compute the theoretical isotopic pattern with enviPat::isopattern and quantify similarity with msentropy::msentropy_similarity. Retain all candidates per ion and score each by mass accuracy, isotopic peak distribution similarity, and adduct-form evidence before summarizing annotation multiplicity.

## Related tools

- **SMAnalyst** (Integrated R/Shiny platform whose Metabolite Annotation module executes the adduct/isotope identification and multi-evidence scoring workflow)
- **MetaboCoreUtils** (Provides isotopologues (isotopic peak-pair detection within ppm tolerance) and mz2mass (m/z-to-neutral-mass conversion over the adduct list))
- **enviPat** (check_ded removes chemically impossible adduct assignments; isopattern computes theoretical isotopic patterns for matched formulas)
- **msentropy** (msentropy_similarity quantifies similarity between observed and theoretical isotopic peak patterns)

## Examples

```
install.packages(c("enviPat", "msentropy", "MetaboCoreUtils"))  # key R packages for the SMAnalyst annotation pipeline
```

## Evaluation signals

- On the SMAnalyst mouse brain test dataset (14,260 pixels x 3,044 ions) against the 1269-metabolite self-built LC-MS/MS library, the run must reproduce 669 identified ions with 374 single, 148 double, and 147 multiple annotations
- Secondary concordance check: 91% overlap of single annotations with MSIannotator
- Every retained adduct assignment must pass enviPat::check_ded chemical feasibility (no impossible assignments such as [M+H-H2O]+ on oxygen-free molecules)
- Isotope/adduct peak pairs must pass spatial correlation with the default threshold of 0.5 before downstream matching
- Isotopic-pattern similarity (msentropy) must be computed only for ions whose matched formulas have isotope peaks; annotation_results.csv must contain all candidate matches per ion with per-candidate scores

## Limitations

- Annotation multiplicity is high (147 of 669 identified ions had three or more annotation results), so isomeric/isobaric ambiguity persists and scoring must be used to prioritize candidates
- Coverage depends on the custom library; the built-in workflow relies on user-uploaded or built-in databases (HMDB/KEGG/LIPIDMAPS supported)
- Adduct search is limited to the specified common forms ([M+H]+, [M+K]+, [M+Na]+, [M+NH4]+, [M+H-H2O]+) unless the user supplies a different adduct list; unusual adducts will be missed
- The pipeline requires R >= v4.1.0 with a long dependency list (enviPat, msentropy, MetaboCoreUtils, Spectra, etc.), which can complicate local installation

## Evidence

- [abstract] comprehensive metabolite annotation scoring system: "a comprehensive metabolite annotation scoring system (mass accuracy, isotopic similarity, adduct evidence)"
- [readme] isotope/adduct identification with spatial correlation validation: "Isotope/adduct identification with spatial correlation validation"
- [readme] multi-evidence scoring: "Multi-evidence scoring (mass accuracy, isotope pattern, adduct form)"
- [intro] mass error tolerance: "Set mass error tolerance (ppm)"
