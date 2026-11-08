---
name: metabolomics-feature-table-construction
description: Use when converting eRah deconvolved GC/MS (e.g. GC–TOF EI) chromatograms
  across multiple samples into a compound-by-sample area table, and compounds must
  be grouped across samples (alignment) and missing entries recovered before downstream
  quantification.
when_to_use_negative:
- Input is raw GC/MS chromatograms that have not yet been deconvolved with eRah's
  deconvolveComp — alignment requires the deconvolved MetaboSet, not raw CDF/mzML
  data.
- Data is processed by peak-picking tools such as XCMS centWave, MetAlign, or ChromaTOF,
  which produce fragment-ion peak tables rather than deconvolved compound spectra;
  eRah's alignment groups deconvolved compounds, not ion peaks.
- AMDIS-style workflows processing samples independently without spectral alignment
  — eRah's value is cross-sample spectral alignment that AMDIS lacks.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: erah
  role: Provides alignComp, setAlPar, recMissComp, alignList, and dataList to align
    deconvolved spectra, recover missing compounds, and output the final compound-by-sample
    table.
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which eRah is written; run alignment and recovery functions
    from an R console/script.
  repo: https://github.com/xdomingoal/erah-devel
- name: gcspikelite
  role: Spike-in example dataset used to demo the min.samples = 6 recovery parameter
    in a nine-sample experiment.
merged_aliases:
- metabolomics-feature-table-analysis
merged_alias_records:
- alias: metabolomics-intensity-table-construction
  slug: metabolomics-intensity-table-construction
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:5a09f2f29026ce04f11ec91004d8fdc43743de0296325ab337ac508a2e7a4b52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  - gcspikelite
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-feature-table-construction

## Summary

Constructs a final GC/MS metabolomics feature table by aligning deconvolved compound spectra across samples and recovering missing compound entries, using eRah's alignComp/recMissComp/dataList pipeline in R. Produces a compound-by-sample table with compound names, matching scores, and integrated areas.

## When to use

Use when converting eRah deconvolved GC/MS (e.g. GC–TOF EI) chromatograms across multiple samples into a compound-by-sample area table, and compounds must be grouped across samples (alignment) and missing entries recovered before downstream quantification.

## When NOT to use

- Input is raw GC/MS chromatograms that have not yet been deconvolved with eRah's deconvolveComp — alignment requires the deconvolved MetaboSet, not raw CDF/mzML data.
- Data is processed by peak-picking tools such as XCMS centWave, MetAlign, or ChromaTOF, which produce fragment-ion peak tables rather than deconvolved compound spectra; eRah's alignment groups deconvolved compounds, not ion peaks.
- AMDIS-style workflows processing samples independently without spectral alignment — eRah's value is cross-sample spectral alignment that AMDIS lacks.

## Inputs

- Deconvolved MetaboSet experiment object from eRah's deconvolveComp step (task_002 output)
- Alignment parameters object from setAlPar (min.spectra.cor, max.time.dist, mz.range)
- Spectral library loaded via importMSP (e.g. MassBank custom library, GMD) for later identification

## Outputs

- Aligned compound table with one row per unique aligned compound (AlignId) and its per-sample area, e.g. aligned_compound_table.csv via alignList(ex)
- Recovered compound-by-sample intensity/area table via dataList(ex), with no missing (zero) values for recovered compounds

## How to apply

Run alignment with alignComp(ex, alParameters = ...) after loading the deconvolved MetaboSet; set alignment parameters with setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600) so compounds appearing across the maximum number of samples with least retention time and spectral distance are grouped, without needing retention-index references. Enforce the 0.90 spectral-correlation and 3-second max retention-time drift thresholds so only highly similar deconvolved spectra merge; co-eluting compounds must remain distinct rows. Then run recMissComp() with min.samples = 6 (as in the gcspikelite nine-sample demo) so compounds found in at least that many samples are re-searched in the remaining ones via LAD regression against the compound's mean target spectrum. Retrieve the aligned list with alignList(ex) and the final compound-name/area table with dataList(ex). Validate that the recovered table contains no missing (zero) values for recovered compounds, and that each grouped entry satisfies the alignment criteria.

## Related tools

- **erah** (Provides alignComp, setAlPar, recMissComp, alignList, and dataList to align deconvolved spectra, recover missing compounds, and output the final compound-by-sample table.) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which eRah is written; run alignment and recovery functions from an R console/script.) — https://github.com/xdomingoal/erah-devel
- **gcspikelite** (Spike-in example dataset used to demo the min.samples = 6 recovery parameter in a nine-sample experiment.)

## Examples

```
ex.al.par <- setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600); ex <- alignComp(ex, alParameters = ex.al.par); ex <- recMissComp(ex, min.samples = 6); write.csv(alignList(ex), 'aligned_compound_table.csv'); write.csv(dataList(ex), 'recovered_data_list.csv')
```

## Evaluation signals

- Aligned table exists and contains per-compound AlignId plus per-sample area columns.
- Each grouped compound entry satisfies min.spectra.cor = 0.90 and max.time.dist = 3 s alignment criteria.
- Co-eluting compounds deconvolved upstream remain distinct rows rather than being merged.
- Recovered data table from dataList(ex) contains no missing (zero) values for recovered compounds.
- Quantitative outputs (compound names, matching scores, integrated areas per sample) are present as eRah's expected output table.

## Limitations

- Alignment parameters must be set prior to running alignComp; changing min.spectra.cor or max.time.dist changes which compounds merge.
- After recMissComp, compounds must always be re-identified — identification cannot be skipped post-recovery.
- Compounds absent in some samples may only be retrieved by the optional missing compound recovery step (recMissComp), which must be explicitly invoked.
- AMDIS-like peak-picking approaches provide quantitative variables based on m/z, retention time window, and fragment-ion area, making compound identification the main bottleneck — this skill assumes compound-level spectra instead.
- No explicit version number stated for the eRah package in the article.

## Evidence

- [methods] Alignment is executed with alignComp after setting parameters via setAlPar; alignment groups compounds by clustering within boundaries of maximum retention time drift and minimum spectral similarity: "Alignment is executed with `alignComp()`"
- [methods] Parameters for alignment must be set before alignComp; defaults constrain spectral similarity and retention time drift: "The parameters also have to be set prior to this."
- [methods] Missing compound recovery via recMissComp determines a target spectrum from the mean spectrum of each aligned compound and retrieves the compound chromatographic profile via LAD regression in samples where it is missing: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [abstract] eRah outputs a table with compound names, matching scores and integrated area of compounds for each sample: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [abstract] Missing compound recovery is a core workflow step that must be run after alignment before quantification: "(iv) missing compound recovery"
- [readme] eRah is freely available on CRAN and integrates spectral deconvolution, alignment, and identification in GC/MS untargeted metabolomics: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
