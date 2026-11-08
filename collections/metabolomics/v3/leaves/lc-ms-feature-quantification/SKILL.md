---
name: lc-ms-feature-quantification
description: Use when centroided LC/MS data (mzML/mzXML) from multiple biological
  groups must be converted into a feature table (m/z, retention time, group-wise intensities)
  for comparative metabolomics.
when_to_use_negative:
- Input is already a validated feature table with group annotations and fold-changes
  — proceed directly to statistics rather than re-running feature detection
- Data are acquired in profile mode without centroiding or the study targets MS/MS-based
  compound identification rather than comparative feature quantification
- Samples lack biological replicates or defined groups, making group-wise intensity
  comparison and replicate-reproducibility filtering meaningless
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metaboseek
  role: Web-based GUI platform (built on shiny) that orchestrates feature detection,
    regrouping, filtering, statistics, EIC inspection, and CSV export
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: R package used through Metaboseek to detect and align molecular features from
    LC/MS data and group samples
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Bioconductor package used to load xcms results into Metaboseek as a Feature
    Table
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: CAMERA
  role: Annotation of grouped features integrated with the xcms feature-detection
    module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:afa908c74fb83d9740b114b9d93c52a048f1c015c936b63981d7c5cabeee7d43
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-ms-feature-quantification

## Summary

Quantify and compare LC/MS molecular features across sample groups using Metaboseek's integrated xcms/CAMERA feature-detection workflow, as demonstrated for the WT vs hacl-1 mutant and OP50 vs JW1653-1 diet comparison in C. elegans.

## When to use

Use when centroided LC/MS data (mzML/mzXML) from multiple biological groups must be converted into a feature table (m/z, retention time, group-wise intensities) for comparative metabolomics.

## When NOT to use

- Input is already a validated feature table with group annotations and fold-changes — proceed directly to statistics rather than re-running feature detection
- Data are acquired in profile mode without centroiding or the study targets MS/MS-based compound identification rather than comparative feature quantification
- Samples lack biological replicates or defined groups, making group-wise intensity comparison and replicate-reproducibility filtering meaningless

## Inputs

- Centroided LC/MS data files (mzML or mzXML) from biological replicates of each sample group
- xcms project folder with sample grouping from feature detection
- Feature table in .mskFT format with processing history (optional, from prior xcms run)
- Group labels by condition (e.g., genotype, diet)

## Outputs

- Feature table (.mskFT) with m/z, retention time, and per-sample intensities
- Filtered diet/genotype-dependent feature table (m/z, rt, group intensities, fold-changes) as .csv
- Extracted ion chromatograms (EICs) validating group-dependent features

## How to apply

Load centroided LC/MS files (mzML/mzXML) into Metaboseek via the Load module, importing the xcms project folder so that sample grouping from the feature-detection run is applied; Metaboseek's GUI wraps the `xcms` package to detect and align molecular features and uses `MSnbase` to load results as a Feature Table. Load the feature table in .mskFT format (retaining processing history) and use the Regroup Table tab to assign group labels (e.g., genotype WT/hacl-1 and diet OP50/JW1653-1). Use Feature Table Actions filters to retain features reproducibly detected across biological replicates and exclude features present in blanks. Run group-wise intensity comparisons (statistics module) to find features enriched in one group versus another — e.g., m/z 496.2317 (cyclopropane GPE 13:1) and m/z 570.2674 (cyclopropane GLEA) enriched in hacl-1 worms fed OP50 — then confirm diet- or genotype-dependence with ppm-based EIC inspection in Grouped EICs. Export the filtered table (m/z, rt, group intensities, fold-changes) as a .csv file for downstream analysis.

## Related tools

- **Metaboseek** (Web-based GUI platform (built on shiny) that orchestrates feature detection, regrouping, filtering, statistics, EIC inspection, and CSV export) — https://github.com/mjhelf/Metaboseek
- **xcms** (R package used through Metaboseek to detect and align molecular features from LC/MS data and group samples) — https://github.com/sneumann/xcms
- **MSnbase** (Bioconductor package used to load xcms results into Metaboseek as a Feature Table) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **CAMERA** (Annotation of grouped features integrated with the xcms feature-detection module)

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); Metaboseek::Metaboseek()  # launch GUI, then Load xcms project -> .mskFT feature table -> Regroup by genotype/diet -> export .csv
```

## Evaluation signals

- Exported CSV contains rows for the diagnostic features m/z 496.2317 (cyclopropane GPE 13:1) and m/z 570.2674 (cyclopropane GLEA) with near-zero or absent mean intensities in the JW1653-1-fed hacl-1 group, matching the reported Fig. 5d-f result
- All retained features are reproducibly detected across biological replicates and features present in blanks are excluded
- Feature table preserves processing history and sample grouping inherited from the xcms project folder after import into the .mskFT table
- ppm-based EIC inspection in Grouped EICs confirms each diet-dependent feature is absent or strongly reduced in JW1653-1-fed worms relative to OP50-fed worms within the correct RT window

## Limitations

- Metaboseek facilitates metabolite annotation up to confidence Level 3 (tentative structure or compound class assignments only) for many detected features
- Many detected features remain unidentified; the space of yet unidentified biogenic small molecules is large
- Metaboseek is under active development with features constantly added, so module behavior may change between versions (see the download page change log)

## Evidence

- [intro] Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation): "Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation)"
- [methods] graphical user interface to set up data analysis with the `xcms` package: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [methods] You can then load xcms results into the app as a "Feature Table" (using `xcms` and `MSnbase` packages: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [abstract] which uncovers more than 200 previously uncharacterized α-oxidation-dependent metabolites in an untargeted comparison of wildtype and α-oxidation-defective hacl-1 mutants: "which uncovers more than 200 previously uncharacterized α-oxidation-dependent metabolites in an untargeted comparison of wildtype and α-oxidation-defective hacl-1 mutants"
- [readme] It is built on functions from the xcms package, and provides a web-based graphical user interface built with shiny: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
