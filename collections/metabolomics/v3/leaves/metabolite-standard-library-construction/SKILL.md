---
name: metabolite-standard-library-construction
description: Use when authentic metabolite standards have been run in positive and
  negative ESI modes as mzML files and you need to construct an in-house RT/m/z library
  without collecting new MS/MS spectra.
when_to_use_negative:
- You need to collect or process redundant MS/MS spectra for library construction
  — metScribeR focuses on MS1 and RT data and does not use MS/MS for library building
- Your goal is untargeted feature detection on experimental (non-standard) samples
  — use targeted peak-picking tools like MZmine, El-MAVEN, or MS-DIAL with the exported
  library instead
- Input is already an exported feature table or the library CSV exists — reload the
  storage_object.RDS to resume rather than repeating the initial ~30 min computation
  for 12000 mzML files
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metScribeR
  role: 'R package with Shiny app implementing the full library-construction workflow:
    mzML parsing, EIC extraction, peak finding/filtering, manual review, and CSV export'
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: R dependency installed via devtools used for chromatogram handling in the
    EIC/peak workflow
  repo: https://github.com/ethanbass/chromatographR
- name: mzR
  role: Bioconductor dependency used by metScribeR to parse mzML files
- name: Shiny
  role: UI framework hosting the interactive workflow (upload inputs, tune filters,
    review peaks, export library)
- name: MassBank of North America (MoNA)
  role: External database automatically searched for MS/MS spectra of standards via
    inchiKey; not required for library construction itself
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:1cbaff8b4a99d103250aedb0bb1266e1d842997c783f09fa28bb6eba4aa71ff6
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
  - MassBank of North America (MoNA)
  - R
  - Shiny
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# metabolite-standard-library-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build an in-house LC-MS metabolite reference library (RT + m/z per standard adduct) from positive/negative ESI mzML files using the metScribeR R/Shiny package, which extracts adduct EICs, finds and filters peaks, and supports single-click manual quality review before exporting a CSV library with MoNA MS/MS spectra.

## When to use

Use when authentic metabolite standards have been run in positive and negative ESI modes as mzML files and you need to construct an in-house RT/m/z library without collecting new MS/MS spectra.

## When NOT to use

- You need to collect or process redundant MS/MS spectra for library construction — metScribeR focuses on MS1 and RT data and does not use MS/MS for library building
- Your goal is untargeted feature detection on experimental (non-standard) samples — use targeted peak-picking tools like MZmine, El-MAVEN, or MS-DIAL with the exported library instead
- Input is already an exported feature table or the library CSV exists — reload the storage_object.RDS to resume rather than repeating the initial ~30 min computation for 12000 mzML files

## Inputs

- mzML files from LC-MS runs of authentic metabolite standards in positive and negative ESI modes (convert with MSConvert if needed)
- standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optionally inchiKey, additional_identifiers)
- adduct_df.csv with columns adduct, change_from_neutral, mode (POS or NEG)
- user-defined mzML_parsing_noise_level, eic_mz_tolerance, eic_rt_tolerance, and output directory

## Outputs

- exported_metScribeR_library.csv containing all 'good' adducts with identity, RT, m/z, and peak quality information
- exported_metScribeR_library_with_metrics.csv retaining good, bad, and multimodal/indeterminate adducts with peak metrics
- exported_metScribeR_MoNA_MSMS.csv with MS/MS information retrieved from MoNA
- storage_object_initial.RDS and storage_object.RDS for resuming the Shiny analysis state
- a .png figure of each manually reviewed peak in the Figures directory

## How to apply

Set the R working directory to the folder holding standards_df.csv (columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path) and adduct_df.csv (adduct, change_from_neutral, mode), launch the Shiny app with metScribeR::runMetScribeRShinyApp(), upload both CSVs plus an output directory, and read the noise plot to set mzML_parsing_noise_level below which all MS observations are dropped. On submit, metScribeR parses the mzML files, computes each expected adduct m/z as monoisotopic_mass + change_from_neutral grouped by POS/NEG mode, and extracts EICs within the user-set m/z and RT tolerances, saving storage_object_initial.RDS for restart without recomputation. In the Find Peaks tab, toggle the density filter and data smoothing, then set filtering criteria — the article's benchmark uses at least 7 data points per peak, width at half maximum intensity between 0 s and 15 s, maximum 250 peaks per EIC, and minimum intensity 10,000 (10x the 1000 background noise level). Manually review each passing peak on the Review Results tab, labeling Good Peak, Bad Peak, or Multimodal/Indeterminate, then on View/Export Library click update library, optionally run check for crossed adducts (requiring additional adducts to elute within the user-defined RT tolerance) and search MoNA for MS/MS data, and export the library CSV. Benchmark against manual labels: precision 0.93, recall 0.98, F1 0.95, with 43 FP, 10 FN, and 226 TN across 860 peaks from 743 adducts, and ~10 s of computational and manual effort per standard.

## Related tools

- **metScribeR** (R package with Shiny app implementing the full library-construction workflow: mzML parsing, EIC extraction, peak finding/filtering, manual review, and CSV export) — https://github.com/ncats/metScribeR
- **chromatographR** (R dependency installed via devtools used for chromatogram handling in the EIC/peak workflow) — https://github.com/ethanbass/chromatographR
- **mzR** (Bioconductor dependency used by metScribeR to parse mzML files)
- **Shiny** (UI framework hosting the interactive workflow (upload inputs, tune filters, review peaks, export library))
- **MassBank of North America (MoNA)** (External database automatically searched for MS/MS spectra of standards via inchiKey; not required for library construction itself)

## Examples

```
setwd(system.file('extdata', package = 'metScribeR')); library(metScribeR); runMetScribeRShinyApp()
```

## Evaluation signals

- storage_object_initial.RDS exists on disk and reloads as a metScribeR storage object containing EIC chromatograms for every standards_df row with at least one non-missing mzML file path
- Recomputed filtering performance matches the reported benchmark exactly: precision 0.93, recall 0.98, F1 0.95, FP=43, FN=10, TN=226
- RTs derived by metScribeR correlate with manual RTs at 0.99
- Exported library CSV contains identity, m/z, RT, and peak quality label columns for all good adducts, and each row was reviewed manually or passed the stated filter criteria
- Column names in standards_df and adduct_df are spelled exactly as required (common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path; adduct, change_from_neutral, mode) — incorrect spellings cause an error

## Limitations

- RT is not yet consistently transferable or predictable across different laboratory-specific or method-specific LC-MS conditions, so the library is most valuable within a single laboratory
- MoNA MS/MS lookup is dependent on server speed and can often be slow; metScribeR otherwise does not require MS/MS data
- For large submissions the initial computation takes time (~30 min for 12000 mzML files), and the library table only refreshes after clicking the update library button
- Multimodal/indeterminate peaks are excluded from precision/recall tallies and require manual judgment; peaks failing filters (e.g., intensity below 10x background noise of 1000) are removed without manual review

## Evidence
