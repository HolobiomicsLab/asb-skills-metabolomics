---
name: lcms-eic-extraction
description: Use when building an in-house LC-MS metabolite standard reference library
  from mzML files and per-adduct EICs must be generated at expected m/z values before
  peak finding, peak-quality filtering, or RT assignment.
when_to_use_negative:
- The input is already a processed feature table or peak list (e.g. from MZmine, El-MAVEN,
  or MS-DIAL) — this skill operates on raw mzML standard runs
- The goal is untargeted feature detection across experimental samples rather than
  library construction from authentic standards
- The standards table lacks any non-missing mzML file path for a row, or column names
  are misspelled — metScribeR errors on incorrectly spelled/cased column names
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: R package providing parse_mzML(), create_eic()/import_eic_data(), user_data_input(),
    and save_storage_object() for the EIC-extraction stage of library construction
  repo: https://github.com/ncats/metScribeR
- name: mzR
  role: Bioconductor dependency used to read/parse the mzML files
- name: chromatographR
  role: Dependency installed for chromatogram handling underlying EIC construction
  repo: ethanbass/chromatographR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:5e827624b80d2eca7ffca401b2a0e6ddc1a01255e5ea758b268e09cc58888332
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# lcms-eic-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts per-adduct extracted ion chromatograms (EICs) from positive- and negative-mode mzML files of authentic metabolite standards, using expected m/z values computed from neutral monoisotopic masses plus adduct mass shifts — the first stage of metScribeR's MS1/RT library-construction workflow.

## When to use

Use when building an in-house LC-MS metabolite standard reference library from mzML files and per-adduct EICs must be generated at expected m/z values before peak finding, peak-quality filtering, or RT assignment.

## When NOT to use

- The input is already a processed feature table or peak list (e.g. from MZmine, El-MAVEN, or MS-DIAL) — this skill operates on raw mzML standard runs
- The goal is untargeted feature detection across experimental samples rather than library construction from authentic standards
- The standards table lacks any non-missing mzML file path for a row, or column names are misspelled — metScribeR errors on incorrectly spelled/cased column names

## Inputs

- standards_df.csv with required columns: common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (one row per standard, at least one non-missing mzML path; optional inchiKey, additional_identifiers)
- adduct_df.csv with required columns: adduct, change_from_neutral, mode (POS or NEG)
- Positive- and negative-ESI LC-MS mzML files (one or more per standard, converted e.g. with MSConvert)
- User parameters: mzML_parsing_noise_level, eic_mz_tolerance, eic_rt_tolerance, output directory

## Outputs

- Per-adduct EIC chromatograms stored in the metScribeR storage object
- storage_object_initial.RDS containing the post-EIC state, reloadable to restart downstream peak finding without recomputation

## How to apply

Read standards_df.csv (common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path) and adduct_df.csv (adduct, change_from_neutral, mode), and set the R working directory so relative mzML paths resolve. Compute each expected adduct m/z as the standard's neutral monoisotopic_mass plus the adduct's change_from_neutral, grouped by POS/NEG mode. Parse the mzML files with metScribeR::parse_mzML(), dropping all MS observations below the user-defined mzML_parsing_noise_level chosen from the noise plot, then extract EICs for every standard-adduct pair within the user-set eic_mz_tolerance via metScribeR::create_eic()/import_eic_data(), grouping data by time and summing to form each chromatogram. Save the intermediate state with metScribeR::save_storage_object() as storage_object_initial.RDS so downstream peak finding can be resumed without repeating the long initial computation (~30 min for 12000 mzML files). EIC m/z and RT tolerances should be set based on the LC-MS instrument's ability to confidently separate two signals.

## Related tools

- **metScribeR** (R package providing parse_mzML(), create_eic()/import_eic_data(), user_data_input(), and save_storage_object() for the EIC-extraction stage of library construction) — https://github.com/ncats/metScribeR
- **mzR** (Bioconductor dependency used to read/parse the mzML files)
- **chromatographR** (Dependency installed for chromatogram handling underlying EIC construction) — ethanbass/chromatographR

## Examples

```
library(metScribeR); setwd(system.file('extdata', package='metScribeR')); runMetScribeRShinyApp()
```

## Evaluation signals

- storage_object_initial.RDS exists on disk and reloads as a metScribeR storage object containing EIC chromatograms for every standards_df row with at least one non-missing mzML file path
- Expected adduct m/z values equal monoisotopic_mass + change_from_neutral, grouped by POS/NEG mode as specified in adduct_df.csv
- All MS observations below the supplied mzML_parsing_noise_level are absent from the parsed data used for EICs
- Each stored chromatogram is built by filtering raw data to the adduct m/z window (user-defined tolerance), grouping by time, and summing per adduct
- Runtime sanity check: initial computation scales with submission size (~30 min for 12000 mzML files), so a small example dataset should complete near-instantly

## Limitations

- metScribeR focuses library building on MS1 & RT data; MS2 data is not used or required at this stage
- Example data use relative mzML paths, so the working directory must be set to the extdata folder or path resolution fails
- Column names in standards_df.csv and adduct_df.csv must be identically spelled/cased or the workflow errors
- For large submissions the initial EIC computation is slow (~30 min for 12000 mzML files); save storage_object_initial.RDS to avoid repeating it

## Evidence

- [results] In the EIC-creation stage, metScribeR reads mzML files, removes MS observations below the user-defined noise threshold, calculates expected m/z values for the metabolite standard adducts, filters raw data by m/z with a user-defined tolerance, and groups data by time and sums for each adduct to create a chromatogram.: [evidence span withheld — anchor status: not_found]
