---
name: rt-library-construction-workflow-analysis
description: Use when building an in-house RT/m/z metabolite standard library from
  authentic-standard LC-MS mzML files, or when benchmarking that library-construction
  effort (per-standard time, total minutes, fold reduction) against a manual peak-picking
  baseline.
when_to_use_negative:
- The goal is untargeted feature detection or peak picking on biological samples rather
  than authentic-standard library construction; tools such as MZmine, El-MAVEN, or
  MS-DIAL are better suited (metScribeR integrates with these downstream)
- The goal is MS/MS spectral library building — metScribeR focuses on MS1 and RT data
  and does not use or require MS/MS for library construction (MS/MS retrieval from
  MoNA is optional and slow)
- Retention times must transfer across different laboratory-specific or method-specific
  LC-MS conditions — the article notes RT is not yet consistently transferable or
  predictable across methods, so in-house libraries apply within a single laboratory
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: metScribeR
  role: Core R package/Shiny app that extracts adduct chromatograms, performs peak
    finding and quality filtering, supports manual peak review, MoNA MS/MS lookup,
    and CSV library export
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Runtime environment for metScribeR and for computing the timing benchmark
    (e.g., 603 x 4.5 min manual baseline vs 31 + 72 min metScribeR total) and writing
    timing_benchmark_summary.csv
- name: Shiny
  role: User interface framework hosting the metScribeR workflow (upload inputs, noise
    plot, Find Peaks, Review Results, View/Export Library tabs)
- name: chromatographR
  role: GitHub dependency installed before metScribeR (chromatogram processing support)
  repo: https://github.com/ethanbass/chromatographR
- name: mzR
  role: Bioconductor dependency installed via BiocManager for parsing mzML mass spectrometry
    files
- name: MSConvert
  role: Optional helper to convert vendor mass spectrometry formats to mzML before
    processing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:df5bd7903301f3689ce0037cdf75c0b3bf0cafec5d591829eb30a19c30196c9e
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
  - R
  - Shiny
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rt-library-construction-workflow-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Semiautomated construction of in-house LC-MS metabolite retention-time (RT) and m/z reference libraries using metScribeR, an R package with a Shiny app that processes authentic-standard mzML files (MS1/RT only, no MS/MS required) and includes peak-quality filtering, manual review, and MoNA MS/MS lookup. Includes benchmarking the workflow's processing time against a manual MassHunter-style baseline (reported 26-fold reduction).

## When to use

Use when building an in-house RT/m/z metabolite standard library from authentic-standard LC-MS mzML files, or when benchmarking that library-construction effort (per-standard time, total minutes, fold reduction) against a manual peak-picking baseline.

## When NOT to use

- The goal is untargeted feature detection or peak picking on biological samples rather than authentic-standard library construction; tools such as MZmine, El-MAVEN, or MS-DIAL are better suited (metScribeR integrates with these downstream)
- The goal is MS/MS spectral library building — metScribeR focuses on MS1 and RT data and does not use or require MS/MS for library construction (MS/MS retrieval from MoNA is optional and slow)
- Retention times must transfer across different laboratory-specific or method-specific LC-MS conditions — the article notes RT is not yet consistently transferable or predictable across methods, so in-house libraries apply within a single laboratory

## Inputs

- mzML files from positive- and negative-mode ESI runs of authentic metabolite standards (converted e.g. with MSConvert)
- standards_df.csv/tsv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optional inchiKey, additional_identifiers)
- adduct_df.csv/tsv with columns adduct, change_from_neutral, mode (POS/NEG)
- User-set background noise level (e.g., 1000), m/z tolerance, and RT tolerance
- Reported benchmark figures for timing comparison: 603 IROA compounds, ~31 min automated, ~72 min manual inspection, 4.5 min/compound manual baseline

## Outputs

- exported_metScribeR_library.csv (all 'good' adducts with identity, m/z, RT, peak quality metrics, manual peak quality label, MS/MS info)
- exported_metScribeR_library_with_metrics.csv (all 'good', 'bad', and 'multimodal/indeterminate' adducts plus peak metrics)
- exported_metScribeR_MoNA_MSMS.csv (MS/MS spectra retrieved from MoNA)
- storage_object.RDS / storage_object_initial (saved app state for resuming analysis)
- Per-peak PNG figures from manual review
- timing_benchmark_summary.csv with one row per timing metric (per-standard effort, total automated time, total manual review time, total baseline time, fold reduction)

## How to apply

Install the dependency chain (devtools, chromatographR from GitHub, mzR from BiocManager, then ncats/metScribeR) and launch the Shiny app with metScribeR::runMetScribeRShinyApp(). Prepare standards_df.csv (required columns: common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path; optional inchiKey) and adduct_df.csv (adduct, change_from_neutral, mode POS/NEG), upload both, read the background-noise level from the noise plot (e.g., noise 1000; exclude peaks with intensity < 10,000 = 10x noise), and set m/z and RT tolerances based on the LC-MS method's ability to separate signals. In Find Peaks, toggle the density filter and data smoothing, then apply benchmark filter criteria: at least 7 data points per peak in a single EIC, width at half maximum intensity > 0 s and < 15 s, and a maximum of 250 peaks per EIC. Manually review every filtered peak in Review Results (Good Peak / Bad Peak / Multimodal-Indeterminate), optionally check crossed adducts and search MoNA for MS/MS, and export the library CSV. To reproduce the efficiency benchmark, load the reported figures into an R data frame (603 IROA compounds, ~31 min automated processing, ~72 min visual inspection, ~10 s per standard, 4.5 min/compound manual baseline), compute manual total = 603 x 4.5 min (~2713.5 min), metScribeR total = 31 + 72 min (~103 min), and derive the fold reduction (~26x); validate that the recomputed fold reduction falls between 25 and 27 and per-standard effort rounds to ~10 s, writing one row per metric to timing_benchmark_summary.csv.

## Related tools

- **metScribeR** (Core R package/Shiny app that extracts adduct chromatograms, performs peak finding and quality filtering, supports manual peak review, MoNA MS/MS lookup, and CSV library export) — https://github.com/ncats/metScribeR
- **R** (Runtime environment for metScribeR and for computing the timing benchmark (e.g., 603 x 4.5 min manual baseline vs 31 + 72 min metScribeR total) and writing timing_benchmark_summary.csv)
- **Shiny** (User interface framework hosting the metScribeR workflow (upload inputs, noise plot, Find Peaks, Review Results, View/Export Library tabs))
- **chromatographR** (GitHub dependency installed before metScribeR (chromatogram processing support)) — https://github.com/ethanbass/chromatographR
- **mzR** (Bioconductor dependency installed via BiocManager for parsing mzML mass spectrometry files)
- **MSConvert** (Optional helper to convert vendor mass spectrometry formats to mzML before processing)

## Examples

```
devtools::install_github('ncats/metScribeR'); metScribeR::runMetScribeRShinyApp()
```

## Evaluation signals

- Recomputed fold reduction in total processing time (manual 603 x 4.5 min ≈ 2713.5 min vs metScribeR 31 + 72 ≈ 103 min) must fall between 25 and 27 (article reports 26-fold)
- Per-standard combined computational and manual effort must round to approximately 10 s
- Correlation between manual and metScribeR-derived retention times should be approximately 0.99
- All retained peaks satisfy the benchmark filters: ≥7 data points per peak in a single EIC, width at half maximum intensity > 0 s and < 15 s, intensity ≥ 10,000, and ≤ 250 peaks per EIC
- Exported library CSV schema contains identity, m/z, RT, peak quality information, manual peak quality label, and MoNA MS/MS data; poor-quality peaks are filtered out as expected

## Limitations

- Initial automated computation is slow for large submissions (~30 min for 12,000 mzML files); use storage_object_initial to restart without repeating the long initial computation
- MoNA MS/MS search depends on MoNA server speed and can often be slow
- Manual review of every filtered peak is still required (Good Peak / Bad Peak / Multimodal-Indeterminate), contributing ~72 min of visual inspection for 603 standards
- The library must be refreshed with the update library button on the View/Export Library tab before results appear in the displayed table
- metScribeR package version was not stated; no version is given in the repository documentation

## Evidence
