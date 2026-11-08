---
name: compound-database-normalisation
description: Use when preparing metabolite reference databases for m/z annotation
  of MALDI/spatial metabolomics peaks and the raw SDF entries contain inconsistent
  fields, non-natural-product elements, or absent adduct masses.
when_to_use_negative:
- The peaks originate from high-resolution FT-ICR/orbitrap data where METASPACE or
  pySM already provides FDR-controlled annotation
- Targeted metabolites require structural confirmation beyond level 4 MSI isomer reporting
  — pseudo-MS/MS evidence via ms1_id/GNPS spectral libraries is the appropriate route
- The analyte set is lipid-only nomenclature work where RefineLipids, not database
  expansion, is the bottleneck
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: SpaMTP
  role: Provides BuildMZAnnotationIndex(), QueryMZAnnotationIndex(), AdductRules(),
    and AnnotateBigData() to build, query, and apply the normalised compound database
    for m/z annotation
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: R
  role: Implementation language for the normalisation and adduct-expansion pipeline
    within SpaMTP
- name: METASPACE
  role: Online reference database used to benchmark annotation concordance (~90% name
    agreement; HMDB 87%, LipidMaps 93%) of the normalised index
- name: ms1_id
  role: Complementary structural-annotation package whose indexed GNPS spectral libraries
    (gnps_minmz100.pkl, gnps_minmz100_k10.pkl) can supply pseudo-MS/MS evidence for
    candidates retained by the normalised database
  repo: github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:6873cd3706449047f2141a852338df3e8ea9d6af472e4f257875cfefa2f88ddc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - METASPACE
  - R
  - SpaMTP
  - ms1_id
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# compound-database-normalisation

## Summary

Pre-processes raw compound SDF databases (ChEBI, HMDB, GNPS, LIPIDMAPS) into a uniform, element-filtered reference table with 47 pre-computed positive/negative-mode adducts, so that exact-mass m/z annotation in SpaMTP can run against a clean index.

## When to use

Use when preparing metabolite reference databases for m/z annotation of MALDI/spatial metabolomics peaks and the raw SDF entries contain inconsistent fields, non-natural-product elements, or absent adduct masses.

## When NOT to use

- The peaks originate from high-resolution FT-ICR/orbitrap data where METASPACE or pySM already provides FDR-controlled annotation
- Targeted metabolites require structural confirmation beyond level 4 MSI isomer reporting — pseudo-MS/MS evidence via ms1_id/GNPS spectral libraries is the appropriate route
- The analyte set is lipid-only nomenclature work where RefineLipids, not database expansion, is the bottleneck

## Inputs

- Raw compound SDF databases: ChEBI, HMDB, GNPS, LIPIDMAPS
- User-defined ionisation polarity (positive/negative mode)
- User-defined adduct set
- User-defined ppm mass-error tolerance (benchmark: ±3 ppm)

## Outputs

- Uniform normalised compound reference table with N/A and zero-value entries removed
- Table of 47 pre-computed adduct masses per neutral mass entry (positive and negative mode)
- Element-filtered m/z annotation index (C, H, N, S, O, P, Br, F, Na, I, Si only)
- Per-peak annotation results (exact mass, formula, annotation name, InChIKey, ppm difference, adduct type)

## How to apply

Convert the downloaded SDF compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) into a uniform reference table, first removing N/A and zero-value entries. For each neutral mass entry calculate the 47 adducts common in positive and negative ionisation mode and append them to the reference table. Filter the expanded table by the experiment's polarity and adduct set, and by an elemental-composition constraint limited to common natural-product elements (C, H, N, S, O, P, Br, F, Na, I, Si) so that only plausible metabolites remain candidates. The normalised index is then used by SpaMTP's exact-mass matching module, which searches every valid m/z at a user-defined ppm error (±3 ppm as benchmarked on the human lung data) and returns exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type. Summarising all isomers per matched m/z yields a level 4 Metabolomics Standards Initiative match. SpaMTP exposes `BuildMZAnnotationIndex()` to perform this expensive expansion once, after which `QueryMZAnnotationIndex()` can use binary interval search; `AdductRules()` documents the ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior used by the engine.

## Related tools

- **SpaMTP** (Provides BuildMZAnnotationIndex(), QueryMZAnnotationIndex(), AdductRules(), and AnnotateBigData() to build, query, and apply the normalised compound database for m/z annotation) — github.com/GenomicsMachineLearning/SpaMTP
- **R** (Implementation language for the normalisation and adduct-expansion pipeline within SpaMTP)
- **METASPACE** (Online reference database used to benchmark annotation concordance (~90% name agreement; HMDB 87%, LipidMaps 93%) of the normalised index)
- **ms1_id** (Complementary structural-annotation package whose indexed GNPS spectral libraries (gnps_minmz100.pkl, gnps_minmz100_k10.pkl) can supply pseudo-MS/MS evidence for candidates retained by the normalised database) — github.com/Philipbear/ms1_id

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP"); index <- BuildMZAnnotationIndex(); AdductRules()
```

## Evaluation signals

- All N/A and zero-value mass entries must be absent from the final normalised reference table
- Every retained neutral-mass entry carries 47 adduct-derived ion masses spanning both positive and negative mode
- No compound whose formula contains elements outside {C, H, N, S, O, P, Br, F, Na, I, Si} survives the elemental-composition filter
- Running the module on the mouse liver spotted-standards peak list reproduces the deposited reference table (results_spotted.csv) and annotation_test_results.csv
- Benchmarked annotation concordance against METASPACE reaches ~90% name agreement (HMDB 87%, LipidMaps 93%) at the chosen ±3 ppm tolerance

## Limitations

- Annotation is reported as a level 4 Metabolomics Standards Initiative match: all possible isomers per matched m/z are listed without structural disambiguation
- The elemental filter excludes compounds containing elements beyond the common natural-product set, so exotic-element metabolites will be silently dropped
- The vignette does not state a specific SpaMTP version, DOI for the refinement article, or hardware/dependency requirements for the normalisation step
- Paired targeted metabolomics comparison for further refinement is documented as a placeholder only ('To come!')

## Evidence

- [methods] Pre-processing and element filtering of compound databases: [evidence span withheld — anchor status: not_found]
- [methods] Adduct expansion step: [evidence span withheld — anchor status: not_found]
- [methods] Element-composition constraint: [evidence span withheld — anchor status: not_found]
- [methods] ppm tolerance benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Index build and query design: "`BuildMZAnnotationIndex()` performs the expensive expansion once"
- [results] Concordance benchmark vs METASPACE: [evidence span withheld — anchor status: not_found]
- [intro] Annotation capability gap motivating the pipeline: "Another major component specific to SM analytical pipelines that is absent from almost all current packages (including Cardinal), is the ability to annotate relative mass-to-charge ratio (m/z) values"
- [methods] Level 4 MSI isomer reporting: [evidence span withheld — anchor status: not_found]
- [readme] Adduct rule registry: "`AdductRules()` returns the ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior used by the engine"
