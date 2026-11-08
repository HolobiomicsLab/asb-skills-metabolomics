---
name: absolute-lipid-concentration-quantification
description: Use when lipidomics peak tables (LipidSearch or LIQUID output) contain
  internal lipid standards and the analysis goal is absolute per-lipid concentration
  comparison between sample groups (e.g., monocytes vs. iDCs).
when_to_use_negative:
- The experiment does not include internal lipid standards — the paper states normalization
  providing absolute values is only possible 'in the presence of internal lipid standards'
- Input is a lipidomic experiment hosted on Metabolomics Workbench fetched via do_query()
  — a different ingestion path, though the same app supports it
- The analysis requires lipid identification from raw spectra; ADViSELipidomics consumes
  LipidSearch/LIQUID outputs, not raw MS data
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ADViSELipidomics
  role: Shiny app that performs internal-standard-based normalization to absolute
    concentrations and computes per-sample recovery percentages
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Source of vendor output files for lipid identification and quantification
    ingested by ADViSELipidomics
- name: Shiny
  role: Framework in which the ADViSELipidomics application is implemented and launched
  repo: https://github.com/ShinyFabio/ADViSELipidomics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:a78fc6787792e8882e5fb5a8febfde76e194e2222e6770975830d82070038401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# absolute-lipid-concentration-quantification

## Summary

Internal-standard-based normalization of lipidomics data matrices to yield absolute concentration values per lipid and per sample, as implemented in ADViSELipidomics for LipidSearch/LIQUID outputs.

## When to use

Use when lipidomics peak tables (LipidSearch or LIQUID output) contain internal lipid standards and the analysis goal is absolute per-lipid concentration comparison between sample groups (e.g., monocytes vs. iDCs).

## When NOT to use

- The experiment does not include internal lipid standards — the paper states normalization providing absolute values is only possible 'in the presence of internal lipid standards'
- Input is a lipidomic experiment hosted on Metabolomics Workbench fetched via do_query() — a different ingestion path, though the same app supports it
- The analysis requires lipid identification from raw spectra; ADViSELipidomics consumes LipidSearch/LIQUID outputs, not raw MS data

## Inputs

- LipidSearch output files for lipid identification and quantification (Case Study 1 archive: Case_Study_1.zip)
- LIQUID output files (alternative vendor format handled by ADViSELipidomics)
- Sample metadata with internal lipid standard information

## Outputs

- Normalized concentration values per lipid and per sample (absolute concentration matrix)
- Percentage of recovery for each sample
- Differentially abundant lipid table for the two-group comparison

## How to apply

Load the vendor output (e.g., LipidSearch files from Case_Study_1.zip) into the ADViSELipidomics Shiny app preprocessing module, which parses lipid species identifiers using LIPID MAPS classification. Because the experiment includes internal lipid standards, run the internal-standard-based normalization so the data matrix yields normalized concentration values per lipids and samples; the app also computes a percentage of recovery for each sample to make comparisons among samples more accurate. Use the resulting normalized concentration table for downstream group comparisons (e.g., differentially abundant lipids in the monocytes-vs-iDCs two-group design). Validate by reproducing the normalized values reported in Case Study 1 of the article's Supplementary Material. R > 4.0 is required; the app can also be run via Docker at localhost:80.

## Related tools

- **ADViSELipidomics** (Shiny app that performs internal-standard-based normalization to absolute concentrations and computes per-sample recovery percentages) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Source of vendor output files for lipid identification and quantification ingested by ADViSELipidomics)
- **Shiny** (Framework in which the ADViSELipidomics application is implemented and launched) — https://github.com/ShinyFabio/ADViSELipidomics

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Normalized concentration table matches the Case Study 1 results reported in the article's Supplementary Material (matching values per lipids and samples)
- Each sample has a computed percentage of recovery before cross-sample comparison
- Output matrix is keyed by lipid species parsed under LIPID MAPS classification with one absolute concentration value per lipid per sample
- Monocytes-vs-iDCs comparison reproduces the differentially abundant lipids from the case study two-group design

## Limitations

- Absolute quantification is only available when the experiment includes internal lipid standards
- UI layout varies with screen size/resolution; the README suggests adjusting zoom (Ctrl +/- on Windows/Linux, Command +/- on Mac)
- Docker version was tested in Chrome; Firefox can have some problems
- Fresh installation of many compiled R packages can take a long time depending on hardware and OS

## Evidence

- [abstract] internal-standard normalization yields absolute values: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [intro] LIPID MAPS parsing supports analyses: "extracts information by parsing lipid species (using LIPID MAPS
classification) and, together with information available on the samples,
allows performing several exploratory and statistical analyses"
- [supplementary] recovery percentage rationale: [evidence span withheld — anchor status: not_found]
- [readme] Case Study 1 data availability: "If you want to test the software with a LipidSearch output, here we
provide the files used in the Case Study 1. Before use, please extract
the files from the archive."
- [methods] launch command: "library("ADViSELipidomics")
run_ADViSELipidomics()"
