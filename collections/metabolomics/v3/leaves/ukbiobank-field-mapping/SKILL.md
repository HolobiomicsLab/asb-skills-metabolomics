---
name: ukbiobank-field-mapping
description: Use when the input is an exported UK Biobank phenotype field CSV/TSV
  (Table Exporter or ukbconv format) containing NMR metabolomics fields that must
  be mapped to short biomarker names, biomarker measurement QC flags, and sample processing
  flags before analysis.
when_to_use_negative:
- 'The goal is to run remove_technical_variation(): the README states no prefiltering
  of samples or columns should be performed and the full raw export must be passed
  in as-is'
- Input is already an analysis-ready biomarker table keyed by eid/visit_index with
  descriptive column names — re-extraction from field IDs is unnecessary
- 'Data are non-NMR UK Biobank fields; nmr_info only maps the NMR metabolomics fields
  (#23651-23948 range and related resources #3000/#3004)'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3277
- http://edamontology.org/topic_3520
tools:
- name: ukbnmr
  role: Provides extract_biomarkers(), extract_biomarker_qc_flags(), and extract_sample_qc_flags()
    to map raw field-ID columns to analysis-ready biomarker and QC-flag data.frames;
    bundles nmr_info and sample_qc_info lookup tables
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Reads the exported field CSV with fread() and writes extracted tables with
    fwrite()
- name: R
  role: Analysis environment in which the ukbnmr extraction and downstream per-covariate
    lm() R-squared computations are run
- name: ukbtools
  role: Alternative upstream processing of pre-RAP ukbconv exports so they are compatible
    with the ukbnmr extraction functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:af54add73a274191243bbb0b3a1519fa2589412830c0f796e7182dc6a9465591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - MASS
  - R
  - data.table
  - ukbnmr
  - ukbtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3277
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ukbiobank-field-mapping

## Summary

Maps raw UK Biobank phenotype field exports (columns named p<field_id>_i<visit_index>_a<repeat_index>) to analysis-ready NMR biomarker, biomarker QC flag, and sample processing QC flag tables using the ukbnmr extraction functions, enabling downstream per-biomarker technical-variation analyses.

## When to use

Use when the input is an exported UK Biobank phenotype field CSV/TSV (Table Exporter or ukbconv format) containing NMR metabolomics fields that must be mapped to short biomarker names, biomarker measurement QC flags, and sample processing flags before analysis.

## When NOT to use

- The goal is to run remove_technical_variation(): the README states no prefiltering of samples or columns should be performed and the full raw export must be passed in as-is
- Input is already an analysis-ready biomarker table keyed by eid/visit_index with descriptive column names — re-extraction from field IDs is unnecessary
- Data are non-NMR UK Biobank fields; nmr_info only maps the NMR metabolomics fields (#23651-23948 range and related resources #3000/#3004)

## Inputs

- Exported UK Biobank phenotype field CSV/TSV from the Table Exporter tool on the RAP (columns named p<field_id>_i<visit_index>_a<repeat_index>), or a ukbconv/ukbtools-processed dataset
- ukbnmr package bundled data.frames nmr_info (biomarker-to-field mappings) and sample_qc_info (sample processing mappings)

## Outputs

- nmr data.frame with one column per NMR biomarker (short descriptive names), rows keyed by eid and visit_index
- biomarker_qc_flags data.frame with one column per biomarker; multi-flags separated by '; '
- sample_qc_flags data.frame with one column per NMR sample processing/QC flag
- CSV exports, e.g. nmr_biomarker_data.csv, nmr_biomarker_qc_flags.csv, nmr_sample_qc_flags.csv, and a biomarker x covariate R-squared table (variance_explained_r2.csv)

## How to apply

Load the exported field data with data.table::fread, then call ukbnmr::extract_biomarkers(), extract_biomarker_qc_flags(), and extract_sample_qc_flags() to produce data.frames with one column per biomarker (short descriptive names) or per QC flag, keyed by 'eid' and 'visit_index' (0 = baseline 2006-2010, 1 = first repeat assessment 2012-2013). Consult the bundled nmr_info data.frame for biomarker-to-field-ID mappings and descriptions, and sample_qc_info for sample processing column mappings. Save each table with data.table::fwrite for reuse; the extracted biomarker and sample-QC-flag tables then form the technical covariate matrix (plate row A-H, column 1-12, shipment batch, spectrometer, degradation time, measurement timestamps) used to fit per-biomarker linear models and record R-squared.

## Related tools

- **ukbnmr** (Provides extract_biomarkers(), extract_biomarker_qc_flags(), and extract_sample_qc_flags() to map raw field-ID columns to analysis-ready biomarker and QC-flag data.frames; bundles nmr_info and sample_qc_info lookup tables) — https://github.com/sritchie73/ukbnmr
- **data.table** (Reads the exported field CSV with fread() and writes extracted tables with fwrite())
- **R** (Analysis environment in which the ukbnmr extraction and downstream per-covariate lm() R-squared computations are run)
- **ukbtools** (Alternative upstream processing of pre-RAP ukbconv exports so they are compatible with the ukbnmr extraction functions)

## Examples

```
library(ukbnmr); library(data.table); exported <- fread("exported_ukbiobank_phenotype_data.csv"); nmr <- extract_biomarkers(exported); sample_qc_flags <- extract_sample_qc_flags(exported)
```

## Evaluation signals

- All 249 biomarker columns present in the extracted nmr table, each mappable to a UK Biobank field ID via nmr_info
- Rows uniquely identifiable by the combination of 'eid' and 'visit_index' (0 or 1) with no duplicates
- Biomarker QC flag cells with no flags contain NA and multi-flag cells contain flags joined by '; '
- Reproduced summary statistics from per-covariate lm() R-squared: median max R-squared ~1.5% across biomarkers, 22/249 biomarkers with >=5% variance explained, alanine ~33% (within-spectrometer drift), histidine ~15% (sample degradation time)
- Round-trip check: every column name in the raw export matching p<field_id>_i<visit>_a<repeat> is consumed by exactly one extraction function or ignored as a non-NMR field

## Limitations

- Extraction is only for UK Biobank NMR metabolomics data; other phenotype fields are not mapped by nmr_info
- No filtering or treatment of outlier samples is performed by the ukbnmr package; downstream analyses must handle QC-flagged samples themselves and are advised to run sensitivity analyses on inclusion/exclusion
- Only a small fraction of samples (2.6%) have blind duplicates for reproducibility assessment, and some technical variation may not be captured by the recorded covariates

## Evidence

- [readme] Exported field data saved by the Table Exporter tool on the RAP has column names following a naming scheme with the format "p<field_id>_i<visit_index>_a<repeat_index>".: [evidence span withheld — anchor status: not_found]
- [methods] The extract_biomarkers() function extracts from this raw field data a data.frame that contains one column per [NMR biomarker]: "The `extract_biomarkers()` function extracts from this raw field data a `data.frame` that contains one column per [NMR biomarker]"
- [readme] Mappings between biomarker column names and UK Biobank field identifiers, along with detailed descriptions of each biomarker, are provided in the nmr_info data.frame: "Mappings between biomarker column names and UK Biobank field identifiers, along with detailed descriptions of each biomarker, are provided in the `nmr_info` `data.frame`"
- [other] Most biomarkers were robust to technical variation (median 1.5% variance explained by the most correlated technical factor), but 22 of 249 biomarkers had at least 5% of variance explained: [evidence span withheld — anchor status: not_found]
- [discussion] Nightingale Health Plc. has flagged some non-missing samples and biomarker measurements with QC flags (see UK Biobank fields #23651–23655: "Nightingale Health Plc. has flagged some non-missing samples and biomarker measurements with QC flags (see UK Biobank fields #23651–23655"
