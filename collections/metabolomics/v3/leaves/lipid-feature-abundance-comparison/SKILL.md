---
name: lipid-feature-abundance-comparison
description: Use when LC/MS feature tables from two or more biological groups (e.g.,
  WT vs hacl-1 mutant on OP50 vs cyclopropane-deficient JW1653-1 diets) must be compared
  to find features enriched in one genotype that additionally depend on the bacterial
  diet.
when_to_use_negative:
- Input is already a filtered, regrouped feature table with diet/genotype labels assigned
  and blanks removed — proceed directly to statistical comparison instead
- The question is structural annotation or molecular formula prediction of features
  rather than group-wise abundance comparison (use Metaboseek's SIRIUS/formula modules
  instead)
- Data are unprocessed profile-mode raw files with no xcms feature detection run —
  feature detection must precede this comparison workflow
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Metaboseek
  role: Web-based GUI platform for loading xcms feature tables, regrouping by genotype/diet,
    filtering features, group-wise statistical comparison, grouped EIC inspection,
    and CSV export
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Feature detection and alignment of molecular features from LC/MS data, and
    feature grouping with CAMERA annotation
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used with xcms to load xcms results into Metaboseek as a Feature Table
- name: CAMERA
  role: Annotation of feature grouping during xcms processing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:0036db603899bcdc41cc0a632a29ab2ab7cad7690370881ef8d88adf89bfd2ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - CAMERA
  - MSnbase
  - Metaboseek
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lipid-feature-abundance-comparison

## Summary

A comparative metabolomics workflow for detecting lipid-derived LC/MS molecular features whose abundance depends on both genotype and diet, exemplified by diet-dependent cyclopropane N-acyl GPEs accumulating in hacl-1 C. elegans mutants (e.g., GPE 13:1, m/z 496.2317) using Metaboseek.

## When to use

Use when LC/MS feature tables from two or more biological groups (e.g., WT vs hacl-1 mutant on OP50 vs cyclopropane-deficient JW1653-1 diets) must be compared to find features enriched in one genotype that additionally depend on the bacterial diet.

## When NOT to use

- Input is already a filtered, regrouped feature table with diet/genotype labels assigned and blanks removed — proceed directly to statistical comparison instead
- The question is structural annotation or molecular formula prediction of features rather than group-wise abundance comparison (use Metaboseek's SIRIUS/formula modules instead)
- Data are unprocessed profile-mode raw files with no xcms feature detection run — feature detection must precede this comparison workflow

## Inputs

- Centroided LC/MS data files (mzML/mzXML) from biological replicates of each genotype/diet group
- xcms project folder with feature detection and grouping results (CAMERA annotation)
- Feature table in .mskFT format with retained processing history
- Blank injection files for blank-feature exclusion

## Outputs

- Filtered diet-dependent feature table (.csv) with m/z, retention time, group intensities, and fold-changes
- Grouped EIC traces confirming presence/absence of target features (e.g., m/z 496.2317, m/z 570.2674) per diet group

## How to apply

Load centroided LC/MS data (mzML/mzXML) from all worm/diet groups into Metaboseek via the Load module, importing the xcms project folder so feature-detection sample grouping is applied; xcms with CAMERA annotation performs feature detection and grouping, and results are converted to a Feature Table with xcms and MSnbase. Load the feature table in .mskFT format (retaining processing history) and use the Regroup Table tab to assign group labels by genotype (WT, hacl-1) and diet (OP50, JW1653-1). In Feature Table Actions, retain features reproducibly detected across biological replicates and exclude features present in blanks, then run group-wise intensity comparison to identify hacl-1-enriched, diet-dependent features such as cyclopropane GPE 13:1 (m/z 496.2317) and cyclopropane GLEA (m/z 570.2674). Confirm each diet-dependent feature by RT-window and ppm-based EIC inspection in Grouped EICs, verifying absence or strong reduction in JW1653-1-fed versus OP50-fed worms, then export the filtered table (m/z, rt, group intensities, fold-changes) as .csv.

## Related tools

- **Metaboseek** (Web-based GUI platform for loading xcms feature tables, regrouping by genotype/diet, filtering features, group-wise statistical comparison, grouped EIC inspection, and CSV export) — https://github.com/mjhelf/Metaboseek
- **xcms** (Feature detection and alignment of molecular features from LC/MS data, and feature grouping with CAMERA annotation) — https://github.com/sneumann/xcms
- **MSnbase** (Used with xcms to load xcms results into Metaboseek as a Feature Table)
- **CAMERA** (Annotation of feature grouping during xcms processing)

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); Metaboseek::runMetaboseek()
```

## Evaluation signals

- Exported CSV contains rows for m/z 496.2317 (cyclopropane GPE 13:1) and m/z 570.2674 (cyclopropane GLEA) with near-zero or absent mean intensities in the JW1653-1-fed hacl-1 group, matching the reported Fig. 5d–f result
- Retained features are reproducibly detected across biological replicates within each genotype/diet group
- Features present in blank injections are excluded from the final table
- Grouped EICs (RT window and ppm-based) confirm strong reduction or absence of diet-dependent features in JW1653-1-fed worms relative to OP50-fed worms
- The exported table preserves m/z, rt, group intensities, and fold-changes per group and matches the .mskFT processing history

## Limitations

- Metabolite annotations are limited to confidence Level 3 (tentative structure or compound class assignments only)
- Many detected features remain unidentified; the space of yet unidentified biogenic small molecules is large
- Diet-dependence inference is specific to the bacterial strains compared (OP50 vs cyclopropane-deficient JW1653-1); features abolished on JW1653-1 are attributed to bacterial cyclopropane fatty acids only by this comparison

## Evidence

- [other] Load module, importing the xcms project folder so sample grouping from the feature detection run is applied: [evidence span withheld — anchor status: not_found]
- [results] production of the most abundant hacl-1-enriched N-acyl GPEs abolished on JW1653-1: [evidence span withheld — anchor status: not_found]
- [discussion] Metaboseek combines more than 60 different modules: "Metaboseek combines more than 60 different modules which tool developers can use as building blocks for specialized data analysis apps"
- [methods] xcms and MSnbase convert xcms results into a Feature Table: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] README: Metaboseek is built on xcms and provides a shiny web GUI: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
