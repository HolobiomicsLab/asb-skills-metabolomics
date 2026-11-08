---
name: mzml-parsing-noise-filtering
description: Use when parsing mzML files of authentic metabolite standards for in-house
  RT/m/z library construction and you need to determine and apply a background noise
  level before extracting adduct EICs.
when_to_use_negative:
- Input is already a feature table or peak list — noise filtering at the mzML observation
  level has no raw signal left to threshold.
- The analysis depends on MS/MS spectra — metScribeR focuses library building on MS1
  & RT data and MS2 data is not used or required.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metScribeR
  role: R package providing parse_mzML() and the Shiny app where the noise level is
    chosen and applied during initial data submission
  repo: https://github.com/ncats/metScribeR
- name: mzR
  role: Bioconductor dependency used to read mzML mass spectrometry files
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: dependency installed alongside metScribeR for chromatogram handling
  repo: https://github.com/ncats/metScribeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:40a39b8919402455fe0a37d6b66b0715a98bdf2fd66e116a5db5a5292093dcf9
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
  - R
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mzml-parsing-noise-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes MS observations below a user-defined background noise threshold (mzML_parsing_noise_level) when parsing positive- and negative-mode mzML files, so that subsequent adduct EIC construction in metScribeR is not cluttered by low-intensity noise.

## When to use

Use when parsing mzML files of authentic metabolite standards for in-house RT/m/z library construction and you need to determine and apply a background noise level before extracting adduct EICs.

## When NOT to use

- Input is already a feature table or peak list — noise filtering at the mzML observation level has no raw signal left to threshold.
- The analysis depends on MS/MS spectra — metScribeR focuses library building on MS1 & RT data and MS2 data is not used or required.

## Inputs

- Positive- and negative-mode mzML files referenced by pos_mode_mzML_file_path and neg_mode_mzML_file_path in standards_df.csv
- User-defined mzML_parsing_noise_level selected from the Shiny noise plot
- standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path
- adduct_df.csv with columns adduct, change_from_neutral, mode

## Outputs

- Noise-filtered parsed MS1 data grouped by POS/NEG mode inside the metScribeR storage object
- storage_object_initial.RDS containing per-adduct EIC chromatograms computed from the filtered data

## How to apply

After launching the metScribeR Shiny app, upload standards_df.csv and adduct_df.csv, select an output directory, and inspect the noise plot figure on the input screen to choose the mzML_parsing_noise_level — the level of background noise below which all MS observations are excluded from processing. When submit is clicked, metScribeR::parse_mzML() reads each pos- and neg-mode mzML file and drops every MS observation below this threshold. The rationale is that EIC creation (expected m/z = monoisotopic_mass + adduct change_from_neutral, filtered within eic_mz_tolerance) then sums only signal above the noise floor, and downstream peak-quality filtering builds on it (e.g. in the benchmarking setup, peaks below 10,000 intensity, i.e. 10x a background noise level of 1000, were excluded). Choose the tolerance/noise settings based on the LC-MS instrument's ability to confidently separate two signals; the intermediate state can be saved as storage_object_initial.RDS so this computation is not repeated.

## Related tools

- **metScribeR** (R package providing parse_mzML() and the Shiny app where the noise level is chosen and applied during initial data submission) — https://github.com/ncats/metScribeR
- **mzR** (Bioconductor dependency used to read mzML mass spectrometry files) — https://github.com/ncats/metScribeR
- **chromatographR** (dependency installed alongside metScribeR for chromatogram handling) — https://github.com/ncats/metScribeR

## Examples

```
library(metScribeR); setwd(system.file('extdata', package='metScribeR')); runMetScribeRShinyApp()
```

## Evaluation signals

- storage_object_initial.RDS exists on disk and reloads as a metScribeR storage object containing EIC chromatograms for every standards_df row with at least one non-missing mzML file path.
- No MS observation below the chosen mzML_parsing_noise_level remains in the parsed data used for EIC construction.
- The chosen noise level is justified by the noise plot figure on the input screen (e.g. consistent with a background noise level such as 1000 in the benchmarking data).
- Each standards_df row has a unique common_name with a single monoisotopic_mass and at least one non-missing mzML file; otherwise parsing errors occur.

## Limitations

- For large submissions the initial computation is slow (~30 min for 12000 mzML files); save/reload storage_object_initial.RDS to avoid repeating it.
- The noise threshold is user-defined from visual inspection of the noise plot; a poorly chosen value propagates into all downstream peak finding and quality filtering.
- Other column names, or incorrectly spelled/cased column names in the input CSVs, will cause an error.

## Evidence

- [results] metScribeR reads mzML files, removes MS observations below the user-defined noise threshold: [evidence span withheld — anchor status: not_found]
