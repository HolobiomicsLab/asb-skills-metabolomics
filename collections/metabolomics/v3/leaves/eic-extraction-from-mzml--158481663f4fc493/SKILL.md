---
name: eic-extraction-from-mzml
description: 'Use when starting a metScribeR library-construction run: you have standards_df.csv
  (common_name, monoisotopic_mass, pos/neg_mode_mzML_file_path) and adduct_df.csv
  (adduct, change_from_neutral, mode) and need EICs per adduct before find_eic_peaks.'
when_to_use_negative:
- The input is already a peak table or feature table (e.g. MZmine output) rather than
  raw mzML files needing chromatogram extraction.
- The library workflow depends on MS/MS spectra for identification — metScribeR focuses
  on MS1 and RT data and MS2 is not used or required.
- mzML files are not yet converted from vendor formats; convert with MSConvert first.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: metScribeR
  role: Provides create_eic/import_eic_data and the Shiny app that builds adduct EICs
    from mzML files given standards_df, adduct_df, m/z and RT tolerances, and the
    mzML_parsing_noise_level.
  repo: https://github.com/ncats/metScribeR
- name: mzR
  role: Bioconductor dependency installed with BiocManager::install('mzR') and used
    to parse the .mzML chromatogram data.
- name: chromatographR
  role: R package installed via devtools::install_github('ethanbass/chromatographR')
    as part of the metScribeR environment.
  repo: https://github.com/ethanbass/chromatographR
- name: MSConvert
  role: Recommended for converting vendor LC-MS formats to .mzML before EIC extraction.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:0d5ff0332bd4caa85374a87f88d231a575fe2d90b65bbec38bc0e6eca78f8be0
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# eic-extraction-from-mzml

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build extracted ion chromatograms (EICs) for each metabolite-standard adduct from positive- and negative-ESI mzML files using metScribeR's create_eic/import_eic_data, setting m/z and RT tolerances and a background noise cutoff (mzML_parsing_noise_level, e.g. 1000) so downstream peak finding operates on noise-filtered chromatograms.

## When to use

Use when starting a metScribeR library-construction run: you have standards_df.csv (common_name, monoisotopic_mass, pos/neg_mode_mzML_file_path) and adduct_df.csv (adduct, change_from_neutral, mode) and need EICs per adduct before find_eic_peaks.

## When NOT to use

- The input is already a peak table or feature table (e.g. MZmine output) rather than raw mzML files needing chromatogram extraction.
- The library workflow depends on MS/MS spectra for identification — metScribeR focuses on MS1 and RT data and MS2 is not used or required.
- mzML files are not yet converted from vendor formats; convert with MSConvert first.

## Inputs

- standards_df.csv with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optionally inchiKey, additional_identifiers)
- adduct_df.csv with columns adduct, change_from_neutral, mode (POS/NEG)
- Positive- and negative-ESI LC-MS data in .mzML format (converted via MSConvert)

## Outputs

- Extracted ion chromatogram (EIC) objects for each standard-adduct combination
- A saved metScribeR storage object (storage_object.RDS) capturing post-initial-computation state

## How to apply

Set R's working directory to the folder containing adduct_df.csv, standards_df.csv, and the mzML files so the relative paths in standards_df resolve. In the Shiny app (or via create_eic/import_eic_data), upload both CSVs, read the noise plot to set the background noise level below which MS observations are excluded from parsing (10,000-intensity peaks correspond to a noise level of 1000 in the benchmark), and choose m/z and RT tolerances calibrated to how confidently the LC-MS instrument separates two signals. EICs are built for every adduct row in adduct_df, combining the neutral monoisotopic mass with change_from_neutral and the POS/NEG mode assignment, using mzR to parse the mzML data. Optionally toggle the density filter and data smoothing to prepare chromatograms for peak finding. The resulting EIC object is the direct input to find_eic_peaks; a large submission of 12,000 mzML files takes roughly 30 minutes for this initial computation.

## Related tools

- **metScribeR** (Provides create_eic/import_eic_data and the Shiny app that builds adduct EICs from mzML files given standards_df, adduct_df, m/z and RT tolerances, and the mzML_parsing_noise_level.) — https://github.com/ncats/metScribeR
- **mzR** (Bioconductor dependency installed with BiocManager::install('mzR') and used to parse the .mzML chromatogram data.)
- **chromatographR** (R package installed via devtools::install_github('ethanbass/chromatographR') as part of the metScribeR environment.) — https://github.com/ethanbass/chromatographR
- **MSConvert** (Recommended for converting vendor LC-MS formats to .mzML before EIC extraction.)

## Examples

```
library(metScribeR); setwd('C:/Users/user123/Downloads/metScribeR_extdata_folder'); runMetScribeRShinyApp()
```

## Evaluation signals

- standards_df.csv passes schema check: each row has a unique common_name, one monoisotopic_mass, and at least one non-missing mzML file path, with the required exact column spellings.
- adduct_df.csv contains the three required columns (adduct, change_from_neutral, mode) and every mode value is POS or NEG.
- The chosen mzML_parsing_noise_level matches the noise plot (benchmark used 1000, i.e. 10x below the 10,000 minimum-intensity peak cutoff).
- EIC creation succeeds for the expected number of standard-adduct pairs and completes within the reported scale (~30 min for 12,000 mzML files).
- A storage_object.RDS is produced so the long initial computation can be reloaded without repeating it.

## Limitations

- m/z and RT tolerances must be tuned per LC-MS method; overly loose settings merge signals the instrument cannot confidently separate.
- Example data uses relative mzML paths, so the working directory must be set with setwd() or the paths fail to resolve.
- The initial EIC computation is expensive (~30 min for 12,000 mzML files), though it only needs to be run once if storage_object.RDS is saved.
- MS/MS information is not collected at this stage and MoNA retrieval, when enabled later, depends on server speed and can be slow.

## Evidence
