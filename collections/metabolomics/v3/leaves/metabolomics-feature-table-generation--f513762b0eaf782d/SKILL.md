---
name: metabolomics-feature-table-generation
description: Use when the input is raw LC-MS/MS spectral files (e.g. mzML/mzXML from
  untargeted metabolomics, including pooled-QC SWATH-DIA acquisitions) and the goal
  is a feature_table.csv of MS1 peak features with compound annotations.
when_to_use_negative:
- The input is already a feature table (peak-intensity matrix) rather than raw MS
  spectra — proceed directly to MetaboAnalyst's statistical analysis modules instead.
- The data are targeted metabolomics or non-LC-MS spectra; the asari-based Spectra
  Processing module is built for LC-MS1/MS2 untargeted workflows.
- Only statistical testing, pathway analysis, or dose-response modeling is needed
  and raw spectra processing has already been completed elsewhere.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MetaboAnalyst 6.0
  role: Web platform hosting the Spectra Processing module used to run asari-based
    LC-MS1 peak detection, MS2 deconvolution, and MS2 annotation
- name: asari
  role: Algorithm underlying LC-MS1 peak detection and quantification within the Spectra
    Processing module
- name: MetaboAnalystR 4.0
  role: R package containing the functions underlying the MetaboAnalyst web server,
    enabling local reproduction of the feature table generation workflow
  repo: xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: Companion LC-MS spectra processing package installed alongside MetaboAnalystR
    for feature detection
  repo: xia-lab/OptiLCMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:f04e13b49205f8a21a72fd03f4de657bcaddb4a3de07f43ec1047b6ec1dfe251
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  - asari
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# metabolomics-feature-table-generation

## Summary

Generates LC-MS1 feature tables with named compound identities from raw untargeted metabolomics spectra using MetaboAnalyst 6.0's asari-based Spectra Processing plus DIA/DDA MS2 deconvolution and annotation. It is the raw-spectra-to-feature-table bridge that precedes statistical and functional analysis.

## When to use

Use when the input is raw LC-MS/MS spectral files (e.g. mzML/mzXML from untargeted metabolomics, including pooled-QC SWATH-DIA acquisitions) and the goal is a feature_table.csv of MS1 peak features with compound annotations.

## When NOT to use

- The input is already a feature table (peak-intensity matrix) rather than raw MS spectra — proceed directly to MetaboAnalyst's statistical analysis modules instead.
- The data are targeted metabolomics or non-LC-MS spectra; the asari-based Spectra Processing module is built for LC-MS1/MS2 untargeted workflows.
- Only statistical testing, pathway analysis, or dose-response modeling is needed and raw spectra processing has already been completed elsewhere.

## Inputs

- Raw LC-MS/MS spectral files from untargeted metabolomics samples (case study: islet high/low-glucose exposures)
- Pooled-QC SWATH-DIA MS2 acquisitions for deconvolution and compound identification
- MS2 reference spectra database (~1.5 million spectra) shipped with MetaboAnalystR 4.0

## Outputs

- feature_table.csv listing detected MS1 features with intensities
- Per-feature compound identity annotations (count of features with at least one potential named chemical identity)
- Deconvoluted MS2 spectra with reconstructed fragment-to-precursor associations

## How to apply

Retrieve the deposited raw LC-MS/MS spectral files (in the islet case study: high/low-glucose exposure samples plus pooled-QC SWATH-DIA acquisitions) and run the MetaboAnalyst 6.0 Spectra Processing module with the asari algorithm to perform LC-MS1 peak detection and quantification on the raw spectra. Deconvolute the data-independent acquisition (DIA) MS2 spectra collected from pooled QC samples to reconstruct fragment-to-precursor associations — this deconvolution step is required specifically for DIA data, whereas DDA handling follows the streamlined deconvolution path of the same module. Annotate the deconvoluted MS2 features against the ~1.5 million MS2 reference spectra database (bundled with the ~500,000-entry metabolite-set knowledgebases) to assign potential named chemical identities. Compile the detected feature table (feature_table.csv) and validate it against the reported benchmark: 27,209 detected MS1 features, of which 4,959 carry at least one potential named identity, within tolerance of the deposited reference. Choose between the web-based MetaboAnalyst 6.0 platform and the locally installed MetaboAnalystR 4.0 R package, which is synchronized with the web server so downloaded R command histories reproduce identical results.

## Related tools

- **MetaboAnalyst 6.0** (Web platform hosting the Spectra Processing module used to run asari-based LC-MS1 peak detection, MS2 deconvolution, and MS2 annotation)
- **asari** (Algorithm underlying LC-MS1 peak detection and quantification within the Spectra Processing module)
- **MetaboAnalystR 4.0** (R package containing the functions underlying the MetaboAnalyst web server, enabling local reproduction of the feature table generation workflow) — xia-lab/MetaboAnalystR
- **OptiLCMS** (Companion LC-MS spectra processing package installed alongside MetaboAnalystR for feature detection) — xia-lab/OptiLCMS

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE); devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Regenerated feature_table.csv must match the reported benchmark of 27,209 detected MS1 features within tolerance of the deposited reference
- Exactly 4,959 of the detected features must carry at least one potential named chemical identity, matching the case-study count
- DIA MS2 deconvolution must reconstruct fragment-to-precursor associations from the pooled-QC SWATH acquisitions
- MS2 annotation results should be traceable to lookups against the ~1.5 million MS2 reference spectra database
- Locally produced results should match the MetaboAnalyst web server output, since MetaboAnalystR is synchronized with the server

## Limitations

- R base with version > 3.6.1 is recommended and compatibility of the latest release (v4.0.0) is still under evaluation.
- Due to issues with Latex, some users may only be able to install MetaboAnalystR without vignettes/documentation.
- Metabolome annotations remain an area of active improvement per the authors' future-work statement, so identity coverage may change between versions.
- No changelog was found for the source repository, complicating version-accuracy tracking.

## Evidence

- [abstract] Spectra Processing module supports asari and MS2: "a significantly enhanced Spectra Processing module with support for MS2 data and the asari algorithm"
- [intro] DIA deconvolution rationale: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [methods] Spectra and knowledgebase resources backing annotation: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] MetaboAnalystR synchrony with web server for reproducibility: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server"
- [readme] Benchmark improvement of feature detection and identification: "MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features"
