---
name: c-elegans-bacterial-diet-metabolomics
description: Use when C. elegans LC/MS features (e.g., hacl-1-enriched N-acyl GPEs)
  must be traced to a dietary bacterial origin via an OP50 vs. diet-mutant (JW1653-1)
  strain comparison.
when_to_use_negative:
- The goal is formula prediction or structural filtering of already-annotated features
  (Seven Golden Rules element/valence/H/C filters), not tracing features to the bacterial
  diet
- The dataset lacks matched worm groups fed both the wild-type diet (OP50) and the
  modified bacterial strain (e.g., JW1653-1), so no diet-contrast can be computed
- Input is raw MS/MS spectra for spectral similarity search rather than comparative
  feature-intensity data across biological replicates
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: GUI platform for loading xcms feature tables, regrouping by genotype/diet,
    filtering, statistics, Grouped EIC inspection, and CSV export
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Detects and aligns molecular features from LC/MS data; its project folder
    supplies sample grouping imported into Metaboseek
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used with xcms to load xcms results into Metaboseek as a Feature Table
- name: CAMERA
  role: Feature grouping annotation integrated in the xcms module within Metaboseek
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:8fda81bf7b4e9abfd8efdedac7ff181fd5ceda76ce2339ca78daa0b6ddbaa376
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# c-elegans-bacterial-diet-metabolomics

## Summary

An untargeted comparative metabolomics workflow in Metaboseek that tests whether worm-enriched metabolites derive from the bacterial food source by comparing feature intensities between worms fed OP50 and the cyclopropane-deficient E. coli strain JW1653-1. It was used to show that cyclopropane-containing N-acyl GPEs (e.g., GPE 13:1, m/z 496.2317) accumulating in hacl-1 mutants are food-derived.

## When to use

Use when C. elegans LC/MS features (e.g., hacl-1-enriched N-acyl GPEs) must be traced to a dietary bacterial origin via an OP50 vs. diet-mutant (JW1653-1) strain comparison.

## When NOT to use

- The goal is formula prediction or structural filtering of already-annotated features (Seven Golden Rules element/valence/H/C filters), not tracing features to the bacterial diet
- The dataset lacks matched worm groups fed both the wild-type diet (OP50) and the modified bacterial strain (e.g., JW1653-1), so no diet-contrast can be computed
- Input is raw MS/MS spectra for spectral similarity search rather than comparative feature-intensity data across biological replicates

## Inputs

- Centroided LC/MS data files (mzML/mzXML) from four worm/diet groups: WT and hacl-1 mutants fed OP50 or JW1653-1
- xcms project folder from the feature-detection run (sample grouping)
- Feature table in .mskT format retaining processing history

## Outputs

- List of hacl-1-enriched, diet-dependent features (e.g., m/z 496.2317 GPE 13:1, m/z 570.2674 GLEA) confirmed abolished in JW1653-1-fed worms
- Filtered diet-dependent feature table exported as .csv with m/z, rt, group intensities, and fold-changes
- Grouped EIC plots confirming absence/reduction of diet-dependent features

## How to apply

Load centroided LC/MS data (mzML/mzXML) from the four worm/diet groups (WT and hacl-1 on OP50 and JW1653-1) into Metaboseek by importing the xcms project folder so sample grouping from the feature-detection run (xcms with CAMERA annotation, loaded as a Feature Table via xcms and MSnbase) is applied. Assign group labels by genotype and diet using the Regroup Table tab, then use Feature Table Actions filters to retain features reproducibly detected across biological replicates and exclude blank features. Run group-wise intensity comparisons to identify features that are both hacl-1-enriched and diet-dependent, and confirm diet dependence by RT-window, ppm-based EIC inspection in Grouped EICs — each diet-dependent feature should be absent or strongly reduced in JW1653-1-fed worms relative to OP50-fed worms. Export the filtered table (m/z, rt, group intensities, fold-changes) as .csv; the key biological readout is abolition of cyclopropane GPE 13:1 (m/z 496.2317) and cyclopropane GLEA (m/z 570.2674) on the cyclopropane-deficient diet.

## Related tools

- **Metaboseek** (GUI platform for loading xcms feature tables, regrouping by genotype/diet, filtering, statistics, Grouped EIC inspection, and CSV export) — https://github.com/mjhelf/Metaboseek
- **xcms** (Detects and aligns molecular features from LC/MS data; its project folder supplies sample grouping imported into Metaboseek) — https://github.com/sneumann/xcms
- **MSnbase** (Used with xcms to load xcms results into Metaboseek as a Feature Table)
- **CAMERA** (Feature grouping annotation integrated in the xcms module within Metaboseek)

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- The exported CSV contains rows for m/z 496.2317 (cyclopropane GPE 13:1) and m/z 570.2674 (cyclopropane GLEA) with near-zero or absent mean intensities in the JW1653-1-fed hacl-1 group, matching the reported Fig. 5d-f result
- Retained features are reproducibly detected across biological replicates and blank-derived features are excluded
- Grouped EIC inspection (RT window + ppm tolerance) shows each diet-dependent feature is absent or strongly reduced in JW1653-1-fed worms relative to OP50-fed worms
- Sample grouping from the imported xcms project is correctly reflected after Regroup Table assignment by genotype (WT, hacl-1) and diet (OP50, JW1653-1)
- The .mskT feature table preserves its processing history through the filtering/regrouping steps

## Limitations

- Findings are specific to the JW1653-1 cyclopropane-deficient E. coli diet; whether alpha-oxidation plays a role in mammalian metabolism of cyclopropyl lipids remains to be determined
- Metabolite annotations from this workflow reach at most confidence Level 3 (tentative structure or compound class assignments only)
- Accession IDs or DOIs for the deposited MS raw data (refs 43–45) are not provided in the discussion, limiting data provenance checks

## Evidence

- [other] Comparative HPLC-HRMS of worms fed OP50 vs the cyclopropane-deficient JW1653-1 strain revealed that production of the most abundant hacl-1-enriched N-acyl GPEs (e.g., cyclopropane-containing GPE 13:1, m/z 496.2317) was abolished in JW1653-1-fed worms: [evidence span withheld — anchor status: not_found]
- [intro] Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation): "Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation)"
- [methods] You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages: "load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data. It is built on functions from the xcms package, and provides a web-based graphical user interface built with shiny: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface"
