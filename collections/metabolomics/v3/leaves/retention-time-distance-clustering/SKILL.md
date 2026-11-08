---
name: retention-time-distance-clustering
description: Use when GC–MS untargeted metabolomics samples have been deconvolved
  independently (e.g. via eRah's deconvolveComp) and the same compound must be matched
  across samples to produce one row per unique compound with per-sample areas.
when_to_use_negative:
- Input is raw, undeconvolved GC/MS chromatogram data — alignment requires a deconvolved
  MetaboSet with per-compound spectra
- Only a single sample is present — alignment across samples is meaningless and the
  clustering step cannot group entries
- Spectra must be identified against an internal reference such as retention indices
  — eRah's alignComp clusters on RT distance and spectral similarity without such
  references
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: erah
  role: Provides alignComp, setAlPar, and alignList used to cluster deconvolved compound
    entries across samples by retention-time distance and spectral similarity
  repo: https://github.com/xdomingoal/erah-devel
- name: R
  role: Language in which the eRah alignment workflow (setAlPar, alignComp, alignList)
    is executed
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:195d7ee24058af15ba1509c351d38756b093165935a3703c7778150cc864ff42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xdomingoal/erah-devel
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# retention-time-distance-clustering

## Summary

Groups deconvolved GC–MS compound entries across samples into aligned compounds by clustering within user-defined retention-time drift and spectral-similarity boundaries using eRah's alignComp, enabling cross-sample compound quantification without internal references such as retention indices.

## When to use

Use when GC–MS untargeted metabolomics samples have been deconvolved independently (e.g. via eRah's deconvolveComp) and the same compound must be matched across samples to produce one row per unique compound with per-sample areas.

## When NOT to use

- Input is raw, undeconvolved GC/MS chromatogram data — alignment requires a deconvolved MetaboSet with per-compound spectra
- Only a single sample is present — alignment across samples is meaningless and the clustering step cannot group entries
- Spectra must be identified against an internal reference such as retention indices — eRah's alignComp clusters on RT distance and spectral similarity without such references

## Inputs

- MetaboSet experiment object produced by eRah's deconvolveComp
- Alignment parameter object from setAlPar (min.spectra.cor, max.time.dist, mz.range)

## Outputs

- Aligned compound table (alignList) with one row per unique aligned compound (AlignId) and its area in each sample

## How to apply

Take the MetaboSet experiment object returned by deconvolveComp, create alignment parameters with setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600) and run alignComp(ex, alParameters). eRah computes the Euclidean distance between retention-time distance and spectral similarity for all compounds in the chromatograms, grouping together compounds appearing in the maximum number of samples with the least retention time and spectral distance. Restricting the spectral comparison to m/z 70–600 keeps the correlation focused on informative fragment ions. Compounds absent in some samples can later be recovered with recMissComp, which requires re-identification afterwards. The aligned table is retrieved with alignList(ex).

## Related tools

- **erah** (Provides alignComp, setAlPar, and alignList used to cluster deconvolved compound entries across samples by retention-time distance and spectral similarity) — https://github.com/xdomingoal/erah-devel
- **R** (Language in which the eRah alignment workflow (setAlPar, alignComp, alignList) is executed) — http://CRAN.R-project.org/package=erah

## Examples

```
ex.al.par <- setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600); ex <- alignComp(ex, alParameters = ex.al.par)
```

## Evaluation signals

- The aligned table exists and contains one row per unique aligned compound with an AlignId and per-sample area columns
- Each grouped compound entry satisfies the user-set min.spectra.cor (e.g. 0.90) and max.time.dist (e.g. 3 s) boundaries
- Co-eluting compounds that were deconvolved as distinct entries remain distinct rows rather than being merged during alignment
- Alignments spanning the maximum number of samples show the least retention-time and spectral distance per eRah's Euclidean-distance criterion

## Limitations

- Alignment boundaries are user-defined; overly lenient max.time.dist or min.spectra.cor can merge distinct compounds
- Compounds missing from some samples are not recovered by alignComp itself; a separate recMissComp step is needed and re-identification must follow it
- eRah's alignment does not rely on internal references such as retention indices

## Evidence

- [methods] Alignment is executed with alignComp(): "Alignment is executed with alignComp()"
- [methods] The parameters also have to be set prior to this.: "The parameters also have to be set prior to this."
- [methods] The alignment list can be returned using: alignList(ex): [evidence span withheld — anchor status: not_found]
- [readme] Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics.: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
- [abstract] e Rah aligns compounds by clustering them within user-defined boundaries of maximum retention time drift and minimum spectral similarity: "The quantitative results of eRah are compared to centWave, the peak-picking algorithm implemented in the widely used XCMS package, MetAlign, and ChromaTOF software."
