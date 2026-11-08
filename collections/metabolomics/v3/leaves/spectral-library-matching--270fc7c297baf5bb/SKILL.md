---
name: spectral-library-matching
description: Use when you have aligned deconvolved compounds (AlignID entries from
  erah's alignComp/recMissComp output) and need to assign metabolite names and match
  factors before integrating peak areas for downstream statistics.
when_to_use_negative:
- Your spectral library is only available in NIST native format — erah cannot read
  it (export your own found spectra to MSP instead);
- Input is raw GC/MS profile data (e.g. raw CDF files) rather than an erah-aligned
  MetaboSet object — deconvolution/alignment must be run first;
- You need metabolite identification for LC/MS or direct infusion data — this skill
  is specific to GC-EI-MS deconvolved compound spectra.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: erah
  role: R package providing identifyComp(), idList(), importGMD() and mslib handling
    that performs the spectral library matching on aligned compounds
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Language/runtime in which erah is implemented and executed
- name: Golm Metabolome Database (GMD)
  role: Recommended reference spectral library imported via importGMD() for compound
    matching
- name: MassBank
  role: Custom version bundled with erah as default mslib; also cited as reference
    library for >80.0 match factors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:e9e88ea17b363fff64f32ee051ef6d07cf0c8409e4176179dfc713d269603ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Golm Metabolome Database (GMD)
  - MassBank
  - R
  - erah
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-library-matching

## Summary

Automated metabolite identification for GC/MS untargeted metabolomics: deconvolved compounds are matched against publicly available EI mass spectral libraries (e.g. the Golm Metabolome Database or a bundled MassBank-derived MSP library) and assigned compound names plus similarity match factors.

## When to use

Use when you have aligned deconvolved compounds (AlignID entries from erah's alignComp/recMissComp output) and need to assign metabolite names and match factors before integrating peak areas for downstream statistics.

## When NOT to use

- Your spectral library is only available in NIST native format — erah cannot read it (export your own found spectra to MSP instead);
- Input is raw GC/MS profile data (e.g. raw CDF files) rather than an erah-aligned MetaboSet object — deconvolution/alignment must be run first;
- You need metabolite identification for LC/MS or direct infusion data — this skill is specific to GC-EI-MS deconvolved compound spectra.

## Inputs

- erah MetaboSet experiment object (.rda) post-deconvolution, alignment and missing compound recovery
- MSP-format mass spectral library (Golm Metabolome Database, MassBank custom mslib, Fiehn or HMDB variants; erah cannot read NIST-format libraries)

## Outputs

- Identification table from idList(ex) with compound names, retention times, AlignID and similarity match factors per compound
- Combined per-sample table joining identified compound names/match factors with integrated compound areas (final output of the erah workflow)

## How to apply

Load the MetaboSet experiment object that erah produced after deconvolution (deconvolveComp), alignment (alignComp), and missing compound recovery (recMissComp); import the spectral library in MSP format — the paper recommends the Golm Metabolome Database via importGMD (e.g. GMD_20111121_VAR5_ALK_MSP.txt) rather than the default mslib, because NIST libraries cannot be read by erah. Run identifyComp(ex), which computes the cosine product between each aligned compound's mean deconvolved spectrum and reference library spectra over a set mass range (70–600 m/z for GMD, with trimethylsilyl fragments m/z 73–75 and 147–149 excluded from comparison), yielding a similarity match factor per candidate. If recMissComp was executed, re-identify the compounds after recovery, as the article mandates. Retrieve identifications with idList(ex) to obtain compound names, retention times, AlignID and match factors, then join them with per-sample integrated areas from dataList(ex)/alignList(ex). Treat matches with a similarity match factor above 80.0 as high-quality, as demonstrated in the article's HIAE serum analysis where dysregulated metabolites showed >80.0 similarity to GMD/MassBank references and were subsequently validated by pure standards.

## Related tools

- **erah** (R package providing identifyComp(), idList(), importGMD() and mslib handling that performs the spectral library matching on aligned compounds) — http://CRAN.R-project.org/package=erah
- **R** (Language/runtime in which erah is implemented and executed)
- **Golm Metabolome Database (GMD)** (Recommended reference spectral library imported via importGMD() for compound matching)
- **MassBank** (Custom version bundled with erah as default mslib; also cited as reference library for >80.0 match factors)

## Examples

```
identifyComp(ex); ids <- idList(ex); areas <- dataList(ex); write.csv(merge(ids, areas), 'identification_table.csv')
```

## Evaluation signals

- Retained identifications reach similarity match factors above 80.0, the threshold reported for high-quality matches to reference spectra in the GMD and MassBank
- idList(ex) returns one identification row per aligned compound (AlignID) with non-empty compound names and match factors
- Match factors are computed with TMS fragment m/z 73–75 and 147–149 excluded and over 70–600 m/z, so a spot check against a raw GMD reference spectrum should agree on these exclusions
- If recMissComp was run, identification is re-executed — the idList of the recovered compounds should contain match factors, not NAs
- Final CSV/table contains both compound names/matching scores and integrated area of compounds for each sample, matching the README-described erah output

## Limitations

- erah cannot read NIST-format spectral libraries; libraries must be provided in MSP format (erah can export found spectra to MSP via importMSP-adjacent tooling)
- After recMissComp, compounds must be re-identified or match factors will be stale/missing
- Library-matched identifications putative; article validated significant hits with pure standards and targeted GC-QqQ MS, LC-QqQ and NMR
- The custom bundled MassBank-derived mslib contains a set of ~500 unique compounds — coverage beyond that requires importing GMD or another MSP library
- No explicit version numbers for erah or libraries are stated in the article; track versions externally

## Evidence

- [methods] Identification can be executed by identifyComp, and accessed by idList as follows: "Identification can be executed by identifyComp, and accessed by idList as follows"
- [results] erah cannot read NIST libraries directly: [evidence span withheld — anchor status: not_found]
- [results] high-quality match threshold >80.0: "These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank."
- [readme] erah output includes compound names, matching scores, and areas: "Outputs a table with compound names, matching scores and the integrated area of the compound for each sample."
- [methods] re-identification after missing compound recovery is required: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [methods] recommendation to use the Golm Metabolome Database: "we strongly encourage the use of the Golm Metabolome Database (GMD)"
