---
name: adduct-annotation-library-construction
description: Use when you have authentic metabolite standards run in positive and
  negative ESI mode as mzML files and need to build a within-laboratory RT/m/z library
  without relying on MS/MS spectral data.
when_to_use_negative:
- Your input is already a finished feature table from a targeted peak-picking tool
  such as MZmine, El-MAVEN, or MS-DIAL rather than raw standard mzML files.
- You require MS/MS spectral data as a mandatory part of library construction — metScribeR
  deliberately focuses on MS1 and RT data and does not use or require MS2 for building
  the library.
- You only have a single-ionisation-mode run (pos or neg) for the standards; the workflow
  expects both pos and neg mzML files referenced in standards_df.csv (at least one
  non-missing per row).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: metScribeR
  role: Core R package and Shiny app that extracts adduct chromatograms, finds peaks,
    assesses peak quality, resolves conflicting adducts, and exports the library CSV.
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: Dependency installed alongside metScribeR (via devtools) that supports the
    chromatographic peak-processing backbone.
  repo: https://github.com/ethanbass/chromatographR
- name: mzR
  role: Bioconductor dependency used to parse the mzML files containing standard chromatograms.
- name: MSConvert
  role: Suggested helper to convert vendor LC-MS formats into .mzML before ingestion.
- name: MoNA
  role: External database automatically searched for MS/MS spectra of standards to
    augment the exported library.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:6191684171ea268792ccbf0cc79a2154cba6c3bc7199037cb98f67594a9ab43e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - MSConvert
  - MoNA
  - R
  - Shiny
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# adduct-annotation-library-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Semiautomated construction of in-house LC-MS metabolite reference libraries by extracting adduct chromatograms from positive and negative ESI mzML files, applying peak-quality filters (≥7 data points per EIC peak, width-at-half-maximum >0 s and <15 s, ≤250 peaks per EIC, intensity ≥10,000), resolving the best adduct per standard, and exporting RT, m/z, and quality metrics as a CSV library.

## When to use

Use when you have authentic metabolite standards run in positive and negative ESI mode as mzML files and need to build a within-laboratory RT/m/z library without relying on MS/MS spectral data.

## When NOT to use

- Your input is already a finished feature table from a targeted peak-picking tool such as MZmine, El-MAVEN, or MS-DIAL rather than raw standard mzML files.
- You require MS/MS spectral data as a mandatory part of library construction — metScribeR deliberately focuses on MS1 and RT data and does not use or require MS2 for building the library.
- You only have a single-ionisation-mode run (pos or neg) for the standards; the workflow expects both pos and neg mzML files referenced in standards_df.csv (at least one non-missing per row).

## Inputs

- standards_df.csv (or .tsv) with required columns: common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path; optional inchiKey and additional_identifiers
- adduct_df.csv (or .tsv) with required columns: adduct, change_from_neutral, mode (POS or NEG)
- Positive and negative ESI mzML files for each authentic standard (converted e.g. with MSConvert)
- User-defined background noise level, m/z tolerance, RT tolerance, and peak-filtering parameters

## Outputs

- exported_metScribeR_library.csv containing all 'good' adducts with identity, m/z, RT, and peak quality information
- exported_metScribeR_library_with_metrics.csv including all 'good', 'bad', and 'multimodal/indeterminate' adducts with peak metrics
- exported_metScribeR_MoNA_MSMS.csv with MS/MS information retrieved from MoNA
- storage_object.RDS (resumable session state) and a Figures directory with a .png of each manually reviewed peak

## How to apply

Create a standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, and neg_mode_mzML_file_path, plus an adduct_df.csv mapping each adduct to its change_from_neutral and POS/NEG mode; set the working directory so the relative mzML paths resolve, then launch the metScribeR Shiny app via runMetScribeRShinyApp(). Upload both CSVs, read the noise plot to set the background noise level (e.g. 1000), and choose m/z and RT tolerances appropriate to your LC-MS equipment. On the Find Peaks tab, toggle the density filter and data smoothing, then apply the benchmark filtering criteria: at least 7 data points per peak in a single EIC, width at half maximum intensity greater than 0 s and less than 15 s, a maximum of 250 peaks per EIC, and exclusion of peaks with intensity below 10,000 (10× the background noise level). Manually review each filtered peak (Good Peak / Bad Peak / Multimodal/Indeterminate), then use 'check for crossed adducts' to require additional adducts from a standard to elute within a user-defined RT tolerance of the assigned RT, and export the library with the Export Library csv button.

## Related tools

- **metScribeR** (Core R package and Shiny app that extracts adduct chromatograms, finds peaks, assesses peak quality, resolves conflicting adducts, and exports the library CSV.) — https://github.com/ncats/metScribeR
- **chromatographR** (Dependency installed alongside metScribeR (via devtools) that supports the chromatographic peak-processing backbone.) — https://github.com/ethanbass/chromatographR
- **mzR** (Bioconductor dependency used to parse the mzML files containing standard chromatograms.)
- **MSConvert** (Suggested helper to convert vendor LC-MS formats into .mzML before ingestion.)
- **MoNA** (External database automatically searched for MS/MS spectra of standards to augment the exported library.)

## Examples

```
install.packages('devtools'); devtools::install_github('ethanbass/chromatographR'); BiocManager::install('mzR'); devtools::install_github('ncats/metScribeR'); library(metScribeR); setwd('C:/Users/user123/Downloads/metScribeR_extdata_folder'); runMetScribeRShinyApp()
```

## Evaluation signals

- On the IROA metabolite standards benchmark, the Pearson correlation between manual MassHunter RTs and metScribeR-derived RTs should reach ~0.99 with an RT RMSD of ~0.54 s (~0.18% of the 5 min cycle time).
- exported_metScribeR_library.csv must contain only adducts labelled 'Good Peak' after manual review, and each row must carry identity, m/z, RT, and peak quality columns.
- Every peak retained in the library must satisfy the benchmark filters: ≥7 data points in a single EIC, width at half maximum intensity >0 s and <15 s, ≤250 peaks per EIC, and intensity ≥10,000 (10× the background noise level of 1000).
- For each standard, non-best adducts must elute within the user-defined RT tolerance of the assigned RT after 'check for crossed adducts' is run.
- The storage_object.RDS must reload and resume a previously started analysis without repeating the initial computation.

## Limitations

- Manual review of every filtered peak is still required before library export, so the workflow is semiautomated rather than fully hands-off (~10 s of computational and manual effort per standard).
- Collection of MS/MS information from MoNA is dependent on server speed and can often be slow.
- RT is not yet consistently transferable or predictable across different LC-MS conditions, so the library is only meaningful within a single laboratory's method.
- The initial computation can be slow for large submissions (~30 min for 12000 mzML files).
- metScribeR does not require MS/MS spectral information; if MS2 is essential to your identification strategy, this skill alone is insufficient.
- For large submissions the initial computation will take some time (~30 min for 12000 mzML files).

## Evidence

- [abstract] Benchmark RT accuracy: "showed a correlation of 0.99 between manual and metScribeR-derived RTs"
- [results] Peak-filter criteria: "the number of data points per peak, was set to at least 7 in a single EIC"
- [readme] Manual review step: "Click Good Peak, Bad Peak, and Multimodal/Indeterminate to indicate your confidence in the quality of the peak"
- [results] Crossed-adduct check: "additional adducts from that standard are required to elute within a user-defined tolerance of the assigned RT"
- [readme] Library construction focus: "metScribeR focuses library building on MS1 & RT data, and MS2 data is not used or required for library construction"
- [abstract] Per-standard effort: "metScribeR required approximately 10 s of computational and manual effort per standard"
- [readme] MoNA search speed: "Note collection of MS/MS information is dependent on the MoNA server speed and can often be slow"
- [readme] Large-submission runtime: "this computation will take some time (~30 min for 12000 mzML files)"
