---
name: raw-spectra-mzml-processing
description: Use when raw LC-MS spectra in centroid mzML format (e.g. from a Metabolomics
  Workbench deposit such as ST002796) need MS1 feature detection, alignment, gap filling
  and annotation before quantification or compound identification.
when_to_use_negative:
- Input is already a processed feature table — the article's statistical-analysis
  and functional-interpretation modules, not raw spectral processing, are the right
  entry point.
- The task is MS2 deconvolution or compound identification rather than MS1 feature
  detection — use PerformDDADeconvolution/PerformDIADeconvolution and PerformDBSearchingBatch
  instead.
- Data are not LC-MS spectra (e.g. GC-MS, NMR, or direct-infusion MS1-only datasets
  outside the LC-MS scope of MetaboAnalystR 4.0).
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalystR 4.0
  role: Primary R package providing the auto-optimized LC-MS1 spectra processing workflow
    (peak picking, alignment, gap filling, peak annotation) over centroid mzML input
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: Companion GitHub package installed alongside MetaboAnalystR that underpins
    the auto-optimized feature detection
  repo: https://github.com/xia-lab/OptiLCMS
- name: R
  role: Execution environment (>= 3.6.1) and stats package used for per-feature Pearson
    correlation against dilution ratios
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:687c45e1b9c9dc37b3ed78f925bb4aa43278b52c11957cbd9135496dcd2679e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# raw-spectra-mzml-processing

## Summary

Process raw centroid mzML LC-MS1 spectra through MetaboAnalystR 4.0's auto-optimized feature detection and quantification workflow (peak picking, alignment, gap filling, annotation), producing a complete MS1 feature table ready for quantification assessment or downstream MS2 identification.

## When to use

Use when raw LC-MS spectra in centroid mzML format (e.g. from a Metabolomics Workbench deposit such as ST002796) need MS1 feature detection, alignment, gap filling and annotation before quantification or compound identification.

## When NOT to use

- Input is already a processed feature table — the article's statistical-analysis and functional-interpretation modules, not raw spectral processing, are the right entry point.
- The task is MS2 deconvolution or compound identification rather than MS1 feature detection — use PerformDDADeconvolution/PerformDIADeconvolution and PerformDBSearchingBatch instead.
- Data are not LC-MS spectra (e.g. GC-MS, NMR, or direct-infusion MS1-only datasets outside the LC-MS scope of MetaboAnalystR 4.0).

## Inputs

- Raw LC-MS1 spectra in centroid mzML format (e.g. serial dilution tubes A-M, 3 replicates per tube, from Metabolomics Workbench accession ST002796)
- Expected dilution-ratio vector from the experimental design (11 mixtures plus pure urine/serum samples) for validation

## Outputs

- Complete MS1 feature table (features x samples) across all processed mzML samples
- Per-feature Pearson correlation values plus fraction of features with r > 0.85, exported to dilution_pearson_correlations.csv

## How to apply

Install MetaboAnalystR 4.0 from GitHub (with its OptiLCMS dependency for auto-optimized spectral processing) into an R >= 3.6.1 environment. Load the raw centroid mzML files (e.g. 13 serial-dilution tubes A-M with three replicates each) and run the auto-optimized LC-MS1 workflow covering peak picking, peak alignment, gap filling, and peak annotation to generate a complete feature table across all samples. For quantification-performance validation, compute per-feature Pearson correlations between measured intensities and known dilution ratios using R's stats package. Accept the result when >90% of detected MS1 features show Pearson r > 0.85 against the dilution design, consistent with the MetaboAnalystR 4.0 benchmark on the serum/urine dilution series. The rationale for the auto-optimized path is that it removes the parameter-expertise burden of alternatives such as XCMS, MS-DIAL, MZmine, or asari.

## Related tools

- **MetaboAnalystR 4.0** (Primary R package providing the auto-optimized LC-MS1 spectra processing workflow (peak picking, alignment, gap filling, peak annotation) over centroid mzML input) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (Companion GitHub package installed alongside MetaboAnalystR that underpins the auto-optimized feature detection) — https://github.com/xia-lab/OptiLCMS
- **R** (Execution environment (>= 3.6.1) and stats package used for per-feature Pearson correlation against dilution ratios) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T); devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- More than 90% of detected MS1 features show Pearson correlation coefficients > 0.85 between intensity and dilution ratio across the 13 dilution points.
- Feature table is complete across all samples after alignment and gap filling (no missing gaps in the tube A-M x 3-replicate design).
- Per-feature r values and the summary fraction are written to dilution_pearson_correlations.csv with one row per feature.
- Reproduction matches the MetaboAnalystR 4.0 benchmark claim of >10% more high-quality MS/MS features versus other tools under default settings.

## Limitations

- Auto-optimized defaults were benchmarked against other tools 'under default settings'; heavily customized parameter needs may require manual tuning outside the auto-optimization path.
- Compatibility of the latest package version (v4.0.0) with R versions other than > 3.6.1 is 'under evaluation' per the README, so environment versioning matters.
- Validation relied on a single public deposit (ST002796); performance on other instrument platforms or matrix types is not established by this evidence.

## Evidence

- [abstract] auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing: "an auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing"
- [intro] powerful computational tools have been developed for individual tasks such as XCMS, MS-DIAL, MZmine, and asari for raw spectral processing: "powerful computational tools have been developed for individual tasks such as XCMS, MS-DIAL, MZmine, and asari for raw spectral processing"
- [methods] devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T): "devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)"
- [methods] devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE): "devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)"
- [readme] R base with version > 3.6.1 is recommended. The compatibility of latest version (v4.0.0) is under evaluation.: "R base with version > 3.6.1 is recommended. The compatibility of latest version (v4.0.0) is under evaluation."
