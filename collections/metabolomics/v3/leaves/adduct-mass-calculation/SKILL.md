---
name: adduct-mass-calculation
description: Use when building an in-house RT/m/z metabolite library from mzML files
  of authentic standards and you need expected per-adduct m/z values to target EIC
  extraction before peak finding.
when_to_use_negative:
- Your input is already a peak/feature table (e.g. from MZmine or MS-DIAL) — metScribeR's
  EIC stage targets raw mzML files of authentic standards
- You require MS/MS-based library construction — metScribeR focuses on MS1 and RT
  data and does not use or require MS2 spectra for this step
- Your adduct list lacks accurate change_from_neutral offsets or ionization modes,
  since the m/z calculation is a simple neutral-mass offset and cannot compensate
  for incorrect adduct definitions
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: R package whose user_data_input()/parse_mzML()/create_eic() functions consume
    the standards and adduct tables and compute expected adduct m/z values for EIC
    extraction
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: R package installed alongside metScribeR supporting the chromatogram processing
    downstream of adduct m/z calculation
  repo: https://github.com/ethanbass/chromatographR
- name: mzR
  role: Bioconductor dependency used by metScribeR to parse the mzML files from which
    adduct-ion chromatograms are extracted
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:d2012ac039c0cc7e1c6d12fb4295e0301ff82dec281aed3cac9d91b4c461b7d0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# adduct-mass-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the expected m/z of each metabolite standard adduct by adding the adduct's change_from_neutral to the standard's neutral monoisotopic_mass, grouped by POS/NEG ionization mode, as the basis for EIC extraction in metScribeR.

## When to use

Use when building an in-house RT/m/z metabolite library from mzML files of authentic standards and you need expected per-adduct m/z values to target EIC extraction before peak finding.

## When NOT to use

- Your input is already a peak/feature table (e.g. from MZmine or MS-DIAL) — metScribeR's EIC stage targets raw mzML files of authentic standards
- You require MS/MS-based library construction — metScribeR focuses on MS1 and RT data and does not use or require MS2 spectra for this step
- Your adduct list lacks accurate change_from_neutral offsets or ionization modes, since the m/z calculation is a simple neutral-mass offset and cannot compensate for incorrect adduct definitions

## Inputs

- standards_df.csv (or .tsv) with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path
- adduct_df.csv (or .tsv) with columns adduct, change_from_neutral, mode (POS/NEG)
- Positive- and negative-mode mzML files for the authentic standards

## Outputs

- Expected m/z values for every standard-adduct pair, grouped by POS/NEG mode
- Per-adduct extracted ion chromatograms stored in a metScribeR storage object (storage_object_initial.RDS)

## How to apply

Create an adduct_df.csv with the required columns adduct, change_from_neutral, and mode (POS or NEG), where change_from_neutral is the difference between the adduct and its neutral mass. For each standard-adduct pair, compute the expected m/z as the standard's neutral monoisotopic_mass plus the adduct's change_from_neutral, grouping pairs by POS or NEG mode to match them against the corresponding positive- or negative-mode mzML files. Pass these computed m/z values, together with a user-defined eic_mz_tolerance (chosen based on the LC-MS instrument's ability to confidently separate two signals) and the background-noise-filtered parsed spectra, to metScribeR's EIC-creation step. The resulting per-adduct chromatograms are stored in the storage object (persisted as storage_object_initial.RDS) so downstream peak finding can be restarted without recomputation.

## Related tools

- **metScribeR** (R package whose user_data_input()/parse_mzML()/create_eic() functions consume the standards and adduct tables and compute expected adduct m/z values for EIC extraction) — https://github.com/ncats/metScribeR
- **chromatographR** (R package installed alongside metScribeR supporting the chromatogram processing downstream of adduct m/z calculation) — https://github.com/ethanbass/chromatographR
- **mzR** (Bioconductor dependency used by metScribeR to parse the mzML files from which adduct-ion chromatograms are extracted)

## Examples

```
setwd(system.file('extdata', package = 'metScribeR')); runMetScribeRShinyApp()
```

## Evaluation signals

- For every standards_df row with at least one non-missing mzML file path, the reloaded storage_object_initial.RDS is a metScribeR storage object containing an EIC chromatogram for that standard-adduct pair
- Each computed expected m/z equals the standard's monoisotopic_mass plus the corresponding adduct's change_from_neutral, and pairs are assigned to POS/NEG mode consistently with the mode column
- storage_object_initial.RDS exists on disk and reloads without rerunning the initial (potentially ~30 min for 12000 mzML files) computation
- Only MS observations above the user-defined mzML_parsing_noise_level (read from the noise plot in the Shiny app) contribute to the chromatograms

## Limitations

- Adduct_df column names must be exactly adduct, change_from_neutral, and mode; other or incorrectly spelled/cased column names cause an error
- EIC m/z tolerance (eic_mz_tolerance) must be tuned to the LC-MS instrument's ability to confidently separate two signals; poor tolerance settings yield ambiguous adduct chromatograms
- For large submissions the initial computation is slow (~30 min for 12000 mzML files), though the saved storage object allows restart without repeating it

## Evidence

- [readme] adduct_df column definitions and mode semantics: "change_from_neutral, with the difference between the adduct and its neutral mass; and mode, with either POS or NEG depending on the charge of the indicated ion"
- [readme] Adduct table is required input to the workflow: "Create adduct_df.csv (or .tsv) with the following columns: adduct, with the name of each adduct"
- [results] Adduct chromatogram extraction is the first stage of the metScribeR processing pipeline: "extracting adduct chromatograms, peak finding, assessing peak quality, and identifying conflicting adducts"
- [readme] m/z tolerance must reflect instrument separation capability: "choose an m/z and RT tolerance for creating EICs and distinguishing between adducts. These tolerances should be set based on the ability for the LC-MS equipment to confidently separate two signals"
