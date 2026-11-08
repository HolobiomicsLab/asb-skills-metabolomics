---
name: gc-ms-missing-compound-recovery
description: Use when processing GC-EI MS untargeted metabolomics data with eRah and
  the aligned compound table has missing entries for compounds found in most samples;
  run after alignComp() and before identification/export.
when_to_use_negative:
- When the aligned compound table is already complete with no missing compound-by-sample
  entries — recovery will add nothing.
- When samples are being processed independently (e.g. AMDIS-style) without a prior
  spectral alignment step; recovery requires an aligned MetaboSet from alignComp().
- For peak-picking (fragment-ion) workflows such as XCMS centWave, where the target
  is fragment ion peaks rather than compound spectra.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: erah
  role: R package providing recMissComp(), the missing compound recovery step run
    after alignComp(); recovers missing compounds using mean target spectrum + LAD
    regression
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: execution environment for the erah package and the recovery workflow
  repo: https://github.com/xdomingoal/erah-devel
- name: gcspikelite
  role: Bioconductor example dataset (nine-sample spike-in) used in the article to
    demonstrate recMissComp with min.samples = 6
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:a47001c0f647af07d689dab36df0cbe78f09a0c932889b118d61c3c78d0cbace
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  - gcspikelite
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gc-ms-missing-compound-recovery

## Summary

eRah's post-alignment missing compound recovery: re-searches aligned compounds in samples where deconvolution failed, using the compound's mean target spectrum and LAD regression to fill the final compound-name/area table.

## When to use

Use when processing GC-EI MS untargeted metabolomics data with eRah and the aligned compound table has missing entries for compounds found in most samples; run after alignComp() and before identification/export.

## When NOT to use

- When the aligned compound table is already complete with no missing compound-by-sample entries — recovery will add nothing.
- When samples are being processed independently (e.g. AMDIS-style) without a prior spectral alignment step; recovery requires an aligned MetaboSet from alignComp().
- For peak-picking (fragment-ion) workflows such as XCMS centWave, where the target is fragment ion peaks rather than compound spectra.

## Inputs

- MetaboSet object holding the aligned compound table from erah alignComp()
- GC/MS raw or deconvolved chromatogram data for the aligned samples

## Outputs

- Recovered compound-by-sample intensity/area table with missing entries filled
- MetaboSet object ready for re-identification (identifyComp) and final export via dataList()

## How to apply

Load the MetaboSet object produced by erah's alignComp() alignment step, which holds the aligned compound table across samples. Run recMissComp() with min.samples set so only compounds found in at least that many samples are re-searched in the remaining ones (the article's nine-sample gcspikelite demo used min.samples = 6). Internally, eRah determines a target spectrum from the mean spectrum of each aligned compound, then retrieves the compound chromatographic profile via LAD regression between that target spectrum and a window around the expected elution time in the samples where the compound is missing. Because identification is invalidated by recovery, re-identify compounds afterward by running identifyComp() again, then export the recovered compound-by-sample intensity/area table via dataList().

## Related tools

- **erah** (R package providing recMissComp(), the missing compound recovery step run after alignComp(); recovers missing compounds using mean target spectrum + LAD regression) — http://CRAN.R-project.org/package=erah
- **R** (execution environment for the erah package and the recovery workflow) — https://github.com/xdomingoal/erah-devel
- **gcspikelite** (Bioconductor example dataset (nine-sample spike-in) used in the article to demonstrate recMissComp with min.samples = 6)

## Examples

```
recMissComp(ex, min.samples = 6); identifyComp(ex); dataList(ex)
```

## Evaluation signals

- After recovery, the exported compound-name/area table contains no missing (zero) values for the recovered compounds — the recovery step is designed to eliminate missing entries in the final table.
- Every recovered compound was previously found in at least min.samples samples in the aligned table (i.e. compounds in fewer than min.samples samples should not be re-searched).
- Compounds are re-identified after recovery (identifyComp is re-run) — check that the final idList/alignList reflects post-recovery spectra, not pre-recovery ones.
- Recovery operates on an aligned MetaboSet object; if the input is not an alignComp output, recMissComp should not be run.

## Limitations

- Recovery requires re-identification of compounds afterwards — the step invalidates prior identification results.
- Validated on serum samples analyzed by GC–TOF MS; generalization to other GC/MS platforms or matrices is not demonstrated in the article.
- No explicit version number stated for the erah package in the article; version tracking must be done manually.
- NIST library cannot be imported directly by erah (must be converted to MSP format first), which constrains which libraries can be used for the subsequent identification step.

## Evidence

- [methods] recMissComp parameter description: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [abstract] eRah workflow description: "(iv) missing compound recovery"
- [readme] eRah tool description: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics"
- [abstract] eRah output description: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
