---
name: pairwise-comparative-metabolomics
description: Use when you have centroided LC/MS data (mzML/mzXML) from two sample
  groups — e.g. hacl-1 vs. wildtype C. elegans, or 13C6-leucine-supplemented vs. unlabeled
  controls — and need to identify features enriched in one group or carrying isotope-label
  mass shifts.
when_to_use_negative:
- You only have a single sample group with no labeled or control comparison table
  — pairwise enrichment and Label Finder require two tables to compare
- The analysis goal is MS/MS fragmentation annotation or molecular networking rather
  than feature-level enrichment between conditions
- Data are uncentroided profile spectra not yet processed by feature detection (run
  xcms feature detection/alignment first)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Interactive shiny-based platform hosting the feature table statistics and
    the LabelFinderModule used for pairwise isotope-tracing comparison
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Feature detection and alignment of molecular features from LC/MS data across
    multiple samples, launched from the Metaboseek GUI
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used together with xcms to load xcms results into Metaboseek as a Feature
    Table (mzR-based)
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: LabelFinderModule (Metaboseek)
  role: Pairwise comparison of labeled vs. unlabeled feature tables to identify isotope-labeled
    compounds via mass-shift filtering
  repo: https://github.com/mjhelf/Metaboseek
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:7ec28e62676590fb832e555c80412012b56bf172296bd02d7b96fc265f7a4fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - LabelFinderModule (Metaboseek)
  - MSnbase
  - Metaboseek
  - Metaboseek (LabelFinderModule)
  - xcms
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pairwise-comparative-metabolomics

## Summary

Two-group comparison of LC/MS molecular features (m/z + retention time) to find features enriched in one condition, e.g. mutant vs. wildtype or isotope-labeled vs. unlabeled samples, using Metaboseek's feature-table statistics and Label Finder module.

## When to use

Use when you have centroided LC/MS data (mzML/mzXML) from two sample groups — e.g. hacl-1 vs. wildtype C. elegans, or 13C6-leucine-supplemented vs. unlabeled controls — and need to identify features enriched in one group or carrying isotope-label mass shifts.

## When NOT to use

- You only have a single sample group with no labeled or control comparison table — pairwise enrichment and Label Finder require two tables to compare
- The analysis goal is MS/MS fragmentation annotation or molecular networking rather than feature-level enrichment between conditions
- Data are uncentroided profile spectra not yet processed by feature detection (run xcms feature detection/alignment first)

## Inputs

- Centroided LC/MS raw data files (.mzML or .mzXML)
- Preprocessed feature tables with 'mz' and 'rt' columns for labeled/enriched and control sample groups

## Outputs

- Isotope-enrichment or group-enrichment table exported as .csv listing features with detected mass shifts (e.g. Δ m/z 5.0167 for 13C5) and negative controls
- Annotated feature table of enriched molecular features (m/z, rt)

## How to apply

Process centroided LC/MS files with Metaboseek's integrated xcms module to detect and align molecular features (m/z + rt) across samples, then load the resulting feature table (with required 'mz' and 'rt' columns) into the Metaboseek shiny app. Run the LabelFinderModule in pairwise mode, comparing the labeled/enriched feature table against the control table; for 13C6-leucine tracing, filter output for the 13C5 isotopologue mass shift of Δ m/z 5.0167, which recovers several hundred 13C5-enriched features including iso-branched fatty acids. Inspect specific feature rows (e.g. hacl-1-enriched 13:1 and 15:1 monounsaturated lipids) to confirm absence of the label shift, which indicates these lipids do not derive from branched-chain fatty acid (BCFA) metabolism. Export the enrichment results as .csv via the Save Table function, validating that the output lists the expected label-shifted features and the expected negatives.

## Related tools

- **Metaboseek** (Interactive shiny-based platform hosting the feature table statistics and the LabelFinderModule used for pairwise isotope-tracing comparison) — https://github.com/mjhelf/Metaboseek
- **xcms** (Feature detection and alignment of molecular features from LC/MS data across multiple samples, launched from the Metaboseek GUI) — https://github.com/sneumann/xcms
- **MSnbase** (Used together with xcms to load xcms results into Metaboseek as a Feature Table (mzR-based)) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **LabelFinderModule (Metaboseek)** (Pairwise comparison of labeled vs. unlabeled feature tables to identify isotope-labeled compounds via mass-shift filtering) — https://github.com/mjhelf/Metaboseek

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); Metaboseek::Metaboseek()  # launch app, load feature tables, run LabelFinderModule in pairwise mode (13C6-Leu vs. unlabeled)
```

## Evaluation signals

- Enrichment table contains several hundred features carrying the Δ m/z 5.0167 (13C5) isotopologue shift for 13C6-leucine tracing
- The hacl-1-enriched 13:1 and 15:1 monounsaturated lipid rows show no detected label shift, reproducing the expected negative result
- Feature table schema includes required 'mz' and 'rt' columns before running Label Finder
- Output is exported as .csv via the Save Table function and can be re-loaded without losing feature (m/z, rt) identity

## Limitations

- Label Finder as applied here filters for a specific isotopologue signature (Δ m/z 5.0167 for 13C5); mass shifts outside the specified label pattern will not be recovered
- Molecular features based on m/z + rt combinations typically do not allow unambiguous compound identification, so enriched features require follow-up tandem-MS/structural characterization
- The app could not be deployed on shinyapps.io because of the mzR package dependency; local or Docker installation is required

## Evidence

- [intro] Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation): "Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation)"
- [methods] graphical user interface to set up data analysis with xcms: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [intro] SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds: "SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds"
- [readme] load xcms results into the app as a Feature Table: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] The combination of m/z value and retention time can be defined as a molecular feature: [evidence span withheld — anchor status: external_doc]
