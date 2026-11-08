---
name: metabolomics-experiment-metadata-setup
description: 'Use when starting an eRah GC-MS untargeted metabolomics analysis: you
  have raw chromatogram files (e.g. NetCDF/CDF from gcspikelite) plus sample class/group
  labels and need a MetaboSet experiment object before calling setDecPar or deconvolveComp.'
when_to_use_negative:
- Input is already a MetaboSet object or a downstream erah output (alignment list,
  identified compound table) — re-running metadata setup would discard prior processing
- The data are already processed peak/feature tables (e.g. centWave/XCMS, MetAlign,
  or ChromaTOF output) rather than raw chromatograms, since eRah's pipeline starts
  from raw GC/MS profile data
- The analysis is not GC/MS-based untargeted metabolomics handled by eRah (e.g. LC-MS
  or NMR data outside the eRah workflow)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: erah
  role: Provides createInstrumentalTable, createPhenoTable, and newExp to build the
    MetaboSet experiment container from raw chromatogram files and sample classes
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open-source language in which eRah is written and in which the metadata tables
    and experiment object are constructed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:e1148f46e1edbb7f2465b16646042c7807a00b469d10068cbcbaf7b2b2bde331
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# metabolomics-experiment-metadata-setup

## Summary

Assembles raw GC/MS chromatogram files and their sample annotations into an eRah MetaboSet experiment object — the mandatory container step (instrumental table + phenotype table via newExp) that must exist before any of eRah's five workflow stages (noise filtering, deconvolution, alignment, missing compound recovery, identification) can run.

## When to use

Use when starting an eRah GC-MS untargeted metabolomics analysis: you have raw chromatogram files (e.g. NetCDF/CDF from gcspikelite) plus sample class/group labels and need a MetaboSet experiment object before calling setDecPar or deconvolveComp.

## When NOT to use

- Input is already a MetaboSet object or a downstream erah output (alignment list, identified compound table) — re-running metadata setup would discard prior processing
- The data are already processed peak/feature tables (e.g. centWave/XCMS, MetAlign, or ChromaTOF output) rather than raw chromatograms, since eRah's pipeline starts from raw GC/MS profile data
- The analysis is not GC/MS-based untargeted metabolomics handled by eRah (e.g. LC-MS or NMR data outside the eRah workflow)

## Inputs

- Raw GC/MS chromatogram files in CDF (NetCDF) format, e.g. the nine gcspikelite spike-in samples
- Sample class/group labels (e.g. targets$Group from the spike-in experiment, ordered by FileName)
- R session with the erah package installed (CRAN 'erah' or dev version xdomingoal/erah-devel)

## Outputs

- Instrumental metadata table (from createInstrumentalTable)
- Phenotype/class table (from createPhenoTable)
- erah MetaboSet experiment object (e.g. info = 'DEMO Experiment'), typically saved as .rda

## How to apply

Build the file vector from the raw CDF chromatograms (e.g. list.files(system.file('data', package='gcspikelite'), full.names=TRUE) filtered on 'CDF', yielding the nine spike-in chromatogram files). Call createInstrumentalTable(files) to record file paths/instrumental metadata, then createPhenoTable(files, cls = classes) where classes comes from targets$Group ordered by FileName for the 3 triplicate classes. Instantiate the experiment with newExp(instrumental, phenotype, info = 'DEMO Experiment'); the resulting MetaboSet is the object every later eRah step (setDecPar, deconvolveComp, alignComp, recMissComp, identifyComp) operates on. Rationale: eRah is a five-core-function R pipeline, so a correctly initialized experiment object is the precondition for stage (i) noise filtering and baseline removal and all subsequent stages.

## Related tools

- **erah** (Provides createInstrumentalTable, createPhenoTable, and newExp to build the MetaboSet experiment container from raw chromatogram files and sample classes) — http://CRAN.R-project.org/package=erah
- **R** (Open-source language in which eRah is written and in which the metadata tables and experiment object are constructed)

## Examples

```
instrumental <- createInstrumentalTable(files); phenotype <- createPhenoTable(files, cls); ex <- newExp(instrumental, phenotype, info = 'DEMO Experiment')
```

## Evaluation signals

- The file vector resolves to exactly the expected raw chromatograms (nine CDF files for gcspikelite)
- The resulting MetaboSet contains all 9 samples of the 3 triplicate classes, with sampleInfo classes matching targets$Group ordered by FileName
- sampleInfo(...) and plotChr on the experiment confirm every sample's chromatogram is attached before stage (i) pre-processing
- The MetaboSet is accepted as input by downstream erah functions (setDecPar/deconvolveComp) without error
- save()/load() round-trip of the experiment (e.g. preprocessed_chromatograms.rda) restores a valid MetaboSet

## Limitations

- No specific version numbers are stated for the eRah package, so reproducibility requires recording the installed erah version independently
- Class ordering depends on the FileName ordering of the phenotype source (targets$Group); a mismatched order silently mislabels samples
- Detailed end-to-end processing guidance for some use cases is deferred to supporting methods rather than the main text

## Evidence

- [abstract] Here we introduce e Rah, a free computational tool written in the open language R composed of five core functions: "Here we introduce e Rah, a free computational tool written in the open language R composed of five core functions"
- [methods] example samples from a spike-in experiment designed for interrogating data processing methods from the gcspikelite Bioconductor dataset: "This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [readme] first-time eRah users are pointed to the package manual to get started: [evidence span withheld — anchor status: external_doc]
