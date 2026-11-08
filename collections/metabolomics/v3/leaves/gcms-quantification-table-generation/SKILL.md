---
name: gcms-quantification-table-generation
description: Use when a GC/MS untargeted metabolomics analysis needs to be finalized
  into a compound-by-sample quantification table after deconvolution, alignment, and
  identification, and the workflow must integrate names, match factors, and areas.
when_to_use_negative:
- Input data has not yet been deconvolved and aligned — identification and quantification
  require an aligned experiment object first
- Raw chromatograms are to be exported for external compound identification (e.g.
  NIST) instead of being finalized in-table; use exportMSP-style export of spectra
  to MSP format for NIST software
- The analysis is targeted by pure standards or LC/NMR validation workflows rather
  than GC/MS spectral deconvolution
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: erah
  role: Provides idList, alignList, and dataList functions to assemble the quantification
    table after deconvolution, alignment, and identification
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Language in which eRah runs and where the final table is assembled and written
    (e.g. to CSV)
  repo: https://github.com/xdomingoal/erah-devel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:777fedd9c816cf173e1efa8163da9da80cdca0656dd40a4dee72b87bc770cff6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gcms-quantification-table-generation

## Summary

Generate a quantitative metabolite table from GC/MS untargeted metabolomics by joining eRah compound identifications (idList) with per-sample integrated compound areas (alignList/dataList), producing a final table with compound names, match scores, and areas.

## When to use

Use when a GC/MS untargeted metabolomics analysis needs to be finalized into a compound-by-sample quantification table after deconvolution, alignment, and identification, and the workflow must integrate names, match factors, and areas.

## When NOT to use

- Input data has not yet been deconvolved and aligned — identification and quantification require an aligned experiment object first
- Raw chromatograms are to be exported for external compound identification (e.g. NIST) instead of being finalized in-table; use exportMSP-style export of spectra to MSP format for NIST software
- The analysis is targeted by pure standards or LC/NMR validation workflows rather than GC/MS spectral deconvolution

## Inputs

- eRah MetaboSet experiment object (e.g. 'testPCOS.rda') with aligned and identified compounds
- GC/MS profile data in non-binary readable format (e.g. NetCDF), per sample
- MS library loaded via importMSP (e.g. GMD or the package-included MassBank custom library)
- Aligned compound list from alignList(ex)

## Outputs

- Compound-by-sample quantification table with compound names, match factors, and integrated areas (e.g. identification_table.csv)
- Identification list from idList(ex) with compound names, retention times, AlignID, and match factors

## How to apply

After deconvolution (deconvolveComp), alignment (alignComp), and identification (identifyComp), load the aligned and identified experiment object and retrieve identifications with idList(ex) to obtain compound names, AlignID, retention times, and match factors. Retrieve the aligned compound areas with alignList(ex) and the final integrated areas with dataList(ex). Join the identifications with the per-sample areas on AlignID to produce a table with compound names, match scores, and the integrated area of each compound for every sample. Write this combined table to identification_table.csv. Before finalizing, verify that compounds considered in the table have high-quality identifications (similarity match factor > 80.0 in the paper's report) and re-run identification after any missing-compound recovery (recMissComp) step.

## Related tools

- **erah** (Provides idList, alignList, and dataList functions to assemble the quantification table after deconvolution, alignment, and identification) — http://CRAN.R-project.org/package=erah
- **R** (Language in which eRah runs and where the final table is assembled and written (e.g. to CSV)) — https://github.com/xdomingoal/erah-devel

## Examples

```
ex <- deconvolveComp(...); ex <- alignComp(...); identifyComp(ex); idList(ex); dataList(ex)
```

## Evaluation signals

- The output table must contain one row per identified compound with compound name, AlignID, retention time, match factor, and per-sample integrated areas
- Every compound retained in the final table should have a similarity match factor greater than 80.0 (the paper's high-quality threshold for GMD/MassBank reference spectra)
- Match factor distribution should be reasonable (e.g. most compounds above the 80.0 threshold) rather than dominated by low-scoring hits
- The combined table must not lose compounds that were present in the aligned list but failed identification (these should be either filtered or explicitly noted)
- The same experiment object must be used for identification and area extraction so AlignID keys line up between idList and dataList

## Limitations

- NIST library cannot be read by eRah; libraries must be in a non-binary readable format such as MSP (NIST is only readable with NIST software)
- After running recMissComp (missing compound recovery) it is necessary to re-identify the compounds before building the final table
- The quantification relies on the integrated area of deconvolved compounds; the paper does not state version numbers for eRah, XCMS, MetAlign, or ChromaTOF used for comparison

## Evidence

- [abstract] erah outputs a table with compound names, matching scores and integrated area of compounds for each sample: [evidence span withheld — anchor status: not_found]
- [methods] Identification can be executed by identifyComp, and accessed by idList as follows: "Identification can be executed by identifyComp, and accessed by idList as follows"
- [results] These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank.: "These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank."
- [readme] Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics.: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
- [methods] we always have to re-identify the compounds after executing the missing compound recovery step: "we always have to re-identify the compounds after executing the missing compound recovery step"
