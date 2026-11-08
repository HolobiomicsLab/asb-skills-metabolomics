---
name: metabolomic-output-format-validation
description: Use when a DaDIA.R run (XCMS 3.11.4+) has produced metabolite_intensity_table.csv
  and ms2_spectra.mgf and you must verify MGF parsability, required CSV columns, and
  DDA/DIA source tagging before MS2 spectral matching.
when_to_use_negative:
- The raw DDA and DIA mzXML files have not yet been processed through DaDIA.R — there
  are no exported artifacts to validate
- The inputs are already a curated, annotated metabolite table (post dot-product >
  0.7 annotation) rather than DaDIA.R's raw exports
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DaDIA.R
  role: 'Produces the artifacts under validation: merges DDA/DIA features into metabolite_intensity_table.csv
    and exports ms2_spectra.mgf with source tags'
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Runs centWave peak picking inside DaDIA.R to generate the feature lists that
    populate the merged table; version 3.11.4+ is required
- name: R
  role: Runtime environment (R 4.0+) in which DaDIA.R and the validation checks are
    executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:1cc57f84934950a10e9e56cadcae741d3f60a2a64d50722123fd958a87fcdb46
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metabolomic-output-format-validation

## Summary

Validates the two exported artifacts of a DaDIA.R run — metabolite_intensity_table.csv (merged DDA/DIA feature intensities) and ms2_spectra.mgf (MS2 spectra tagged with DDA/DIA origin) — so downstream metabolite annotation works on well-formed inputs.

## When to use

Use when a DaDIA.R run (XCMS 3.11.4+) has produced metabolite_intensity_table.csv and ms2_spectra.mgf and you must verify MGF parsability, required CSV columns, and DDA/DIA source tagging before MS2 spectral matching.

## When NOT to use

- The raw DDA and DIA mzXML files have not yet been processed through DaDIA.R — there are no exported artifacts to validate
- The inputs are already a curated, annotated metabolite table (post dot-product > 0.7 annotation) rather than DaDIA.R's raw exports

## Inputs

- ms2_spectra.mgf — combined MS2 spectra exported by DaDIA.R with DDA/DIA origin tags in MGF headers
- metabolite_intensity_table.csv — merged DDA/DIA metabolite-intensity table
- DDA and DIA-(SWATH) mzXML files used as DaDIA.R inputs (demo files shipped in the DaDIA GitHub repo)

## Outputs

- Validation report confirming ms2_spectra.mgf parses as valid MGF format
- Confirmation that metabolite_intensity_table.csv exists with the required columns
- DDA/DIA source-split proportions of the exported MS2 spectra (expected ~63% DDA / 37% DIA on urine test data)

## How to apply

After DaDIA.R merges DDA-derived and DIA-derived features (centWave peak-picking in XCMS) and exports ms2_spectra.mgf, first parse the MGF to confirm every spectrum record is valid and carries a DDA or DIA origin tag in its header. Second, check that metabolite_intensity_table.csv exists with the required metabolite-intensity columns, using the unique DDA feature information that DaDIA.R used to align and consolidate cross-mode features. Third, spot-check the DDA/DIA split of exported MS2 spectra against the reported reference proportions from the urine test data (63% DDA / 37% DIA) — large deviations suggest feature-merging or source-attribution errors. Also honor the README constraint that the sample count hardcoded at line 38 of the DaDIA.R script must equal the real number of DIA mzXML files, otherwise the merged table will misattribute sample intensities.

## Related tools

- **DaDIA.R** (Produces the artifacts under validation: merges DDA/DIA features into metabolite_intensity_table.csv and exports ms2_spectra.mgf with source tags) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Runs centWave peak picking inside DaDIA.R to generate the feature lists that populate the merged table; version 3.11.4+ is required)
- **R** (Runtime environment (R 4.0+) in which DaDIA.R and the validation checks are executed)

## Examples

```
Rscript DaDIA.R  # after confirming line 38 sample count matches the number of DIA mzXML files; then parse ms2_spectra.mgf and check metabolite_intensity_table.csv columns
```

## Evaluation signals

- ms2_spectra.mgf parses cleanly as valid MGF format with no malformed spectrum records
- metabolite_intensity_table.csv exists and contains the required metabolite-intensity columns
- DDA/DIA origin tags across exported MS2 spectra match the reported 63% DDA / 37% DIA split for the urine test data
- The sample count at line 38 of the DaDIA.R script equals the actual number of DIA mzXML input files
- Merged table row counts are consistent with DDA features plus rescued features (rescued features averaged 344, ~22% of the 1591 DDA features in the urine study)

## Limitations

- The 63%/37% DDA/DIA MS2 split is a reference from the urine test data; other sample types (e.g., leukemia plasma) may legitimately differ
- The number of samples hardcoded at line 38 of DaDIA.R must match the real number of DIA mzXML files or outputs will be incorrect
- Requires R 4.0+, XCMS development version 3.11.4+, and metaMS 1.25.1; older XCMS versions may not run DaDIA.R

## Evidence
