---
name: spectral-similarity-matching
description: Use when deconvolved GC/MS compound spectra must be compared by similarity
  — to align the same compound across samples (clustering by retention-time drift
  and spectral correlation) or to identify metabolites by spectral library matching.
when_to_use_negative:
- Input is raw GC/MS chromatograms that have not yet been deconvolved — run deconvolveComp
  first, since similarity is computed on reconstructed pure compound spectra, not
  fragment-ion peaks
- Only a NIST library is available for identification — it can only be read with NIST
  software, so export/import an MSP-format library instead
- Input is a peak-picking feature table (m/z value, retention-time window, fragment-ion
  peak area, e.g. from centWave/XCMS or MetAlign) — those quantitative variables are
  not compound spectra and cannot be matched by this workflow
tools:
- name: erah
  role: Provides alignComp/setAlPar for similarity-based cross-sample alignment and
    identifyComp/idList with importMSP for spectral library matching
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which eRah is written and in which the similarity matching
    workflow is executed
  repo: https://github.com/xdomingoal/erah-devel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:eead850a7c5e1af1cee8ac8afeb9ac86603a05889fc04b93fdc4587879a66675
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- spectral-similarity-matrix-computation
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-similarity-matching

## Summary

Matches deconvolved GC-EI MS compound spectra by spectral similarity, both to group the same metabolite across samples during alignment and to identify compounds by spectral library matching against reference EI spectra (e.g. MassBank, GMD). This is eRah's compound-centric alternative to fragment-ion peak picking.

## When to use

Use when deconvolved GC/MS compound spectra must be compared by similarity — to align the same compound across samples (clustering by retention-time drift and spectral correlation) or to identify metabolites by spectral library matching.

## When NOT to use

- Input is raw GC/MS chromatograms that have not yet been deconvolved — run deconvolveComp first, since similarity is computed on reconstructed pure compound spectra, not fragment-ion peaks
- Only a NIST library is available for identification — it can only be read with NIST software, so export/import an MSP-format library instead
- Input is a peak-picking feature table (m/z value, retention-time window, fragment-ion peak area, e.g. from centWave/XCMS or MetAlign) — those quantitative variables are not compound spectra and cannot be matched by this workflow

## Inputs

- MetaboSet experiment object of deconvolved compounds (output of deconvolveComp with setDecPar)
- Alignment parameters object from setAlPar (min.spectra.cor, max.time.dist, mz.range)
- Spectral library in MSP or eRah-readable non-binary format, loaded via importMSP() (e.g. eRah's custom MassBank library, GMD, Fiehn, HMDB)

## Outputs

- Aligned compound table from alignList(ex) with one row per unique aligned compound (AlignId) and per-sample integrated areas (aligned_compound_table.csv)
- Identification list from idList(ex) with compound names and matching scores per sample

## How to apply

For cross-sample alignment, define parameters with setAlPar (e.g. min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600), restricting spectral comparison to the 70–600 m/z window, then run alignComp on the deconvolved MetaboSet object; eRah computes a distance combining retention-time distance and spectral similarity over the chromatograms so that compounds appearing in the most samples with the least RT and spectral distance are grouped together — notably without requiring internal references such as retention indices. Retrieve the grouping with alignList(ex), which yields one row per unique aligned compound (AlignId) with its integrated area per sample. For identification, load a reference library of compound spectra in non-binary readable format with importMSP() (NIST libraries cannot be read directly), run identifyComp, and inspect results with idList; a match factor above ~80.0 against GMD or MassBank reference EI spectra indicates a credible assignment. If missing compound recovery (recMissComp) is executed afterwards, compounds must be re-identified.

## Related tools

- **erah** (Provides alignComp/setAlPar for similarity-based cross-sample alignment and identifyComp/idList with importMSP for spectral library matching) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which eRah is written and in which the similarity matching workflow is executed) — https://github.com/xdomingoal/erah-devel

## Examples

```
ex.al.par <- setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600); ex <- alignComp(ex, alParameters = ex.al.par); al <- alignList(ex)
```

## Evaluation signals

- aligned_compound_table.csv exists and contains per-compound AlignId and per-sample area columns
- Each grouped compound entry satisfies the alignment criteria min.spectra.cor = 0.90 and max.time.dist = 3 s
- Spot-check that co-eluting compounds deconvolved upstream remain distinct rows in the aligned table rather than being merged
- Identified compounds show a high similarity match factor (>80.0) to reference MS spectra in the GMD and MassBank
- After running recMissComp, compounds are re-identified before downstream use of identification scores

## Limitations

- NIST library spectra can only be read with NIST software; eRah instead exports/imports spectra in MSP format
- Alignment relies solely on retention-time drift and spectral similarity boundaries without retention indices, so compounds absent in some samples require the optional recMissComp recovery step
- Compounds must be re-identified after executing the missing compound recovery step

## Evidence

- [methods] Alignment is executed with `alignComp()`: "Alignment is executed with `alignComp()`"
- [methods] The alignment list can be returned using: alignList(ex): [evidence span withheld — anchor status: not_found]
- [methods] Then, we can load the library with the function importMSP(): "Then, we can load the library with the function importMSP()"
- [results] These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank.: "These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank."
- [abstract] e Rah outputs a table with compound names, matching scores and integrated area of compounds for each sample: [evidence span withheld — anchor status: not_found]
- [readme] Outputs a table with compound names, matching scores and the integrated area of the compound for each sample.: "Outputs a table with compound names, matching scores and the integrated area of the compound for each sample."
