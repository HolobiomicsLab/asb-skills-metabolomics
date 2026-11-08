---
name: gc-ms-metabolite-concentration-table-processing
description: Use when you have raw GC/MS metabolomics chromatograms (e.
when_to_use_negative:
- The input is already a validated concentration table with group statistics — re-running
  the full deconvolution/identification pipeline would be redundant.
- Data are not GC/MS (EI) chromatograms; e.g. LC-MS or NMR data, where eRah's deconvolution
  and library matching do not apply.
- The goal is targeted quantification with pure standards rather than untargeted metabolite
  identification from library matching.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: erah
  role: R package that performs deconvolution, alignment, missing compound recovery
    and library-based identification, outputting the compound name/score/area table
    consumed by this skill.
  repo: http://CRAN.R-project.org/package=erah
- name: erah-devel
  role: Development version of eRah installed from GitHub when CRAN release is insufficient;
    documentation and issue tracker for bugs.
  repo: https://github.com/xdomingoal/erah-devel
- name: XCMS
  role: Reference peak-picking algorithm (centWave) used in the article for comparison
    against eRah's quantitative results; not part of the eRah pipeline itself.
- name: MetAlign
  role: Third-party peak-picking tool used in the article for benchmarking eRah's
    quantification; not part of the eRah pipeline.
- name: ChromaTOF
  role: Commercial software (LECO Corp.) used in the article as a benchmark for eRah's
    quantitative results.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:bdb953bae92dc88a95bedd3df63c861d85b2af934c0ce77afc145b32e487b8b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChromaTOF
  - MetAlign
  - R
  - XCMS
  - erah
  - erah-devel
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8484848484848485
n_claims: 33
---

# gc-ms-metabolite-concentration-table-processing

## Summary

Post-process eRah GC/MS untargeted metabolomics output tables (compound names, matching scores, integrated areas per sample) into a normalized concentration matrix and reproduce paper-level statistics such as percent-of-variation and Wilcoxon-Mann-Whitney p-values for dysregulated metabolites.

## When to use

Use when you have raw GC/MS metabolomics chromatograms (e.g. GC-TOF MS serum samples), have already run eRah deconvolution/alignment, and need to turn the resulting compound-by-sample table into relative concentrations plus group-level statistics like percentage-of-variation and rank-sum p-values for dysregulated metabolites.

## When NOT to use

- The input is already a validated concentration table with group statistics — re-running the full deconvolution/identification pipeline would be redundant.
- Data are not GC/MS (EI) chromatograms; e.g. LC-MS or NMR data, where eRah's deconvolution and library matching do not apply.
- The goal is targeted quantification with pure standards rather than untargeted metabolite identification from library matching.

## Inputs

- Raw GC/MS chromatogram files (e.g. GC-TOF MS data from serum/plasma samples, MetaboLights accession MTBLS321)
- eRah experiment object produced by deconvolveComp/alignComp/recMissComp/identifyComp
- eRah putative identifications with relative concentrations per sample (article Supporting File 4)
- Mass spectral reference library in MSP format (bundled MassBank library, GMD, Fiehn, HMDB, or NIST-exported MSP)
- Sample class labels (e.g. 11 HIAE patients vs 14 CTR healthy controls)

## Outputs

- Aligned compound-by-sample table with compound names, matching scores, and integrated areas (from dataList(ex) / alignList(ex))
- Relative concentration matrix for selected metabolites across samples
- Summary table of metabolite name, percentage-of-variation, and Wilcoxon-Mann-Whitney p-value (e.g. table3_metabolite_stats.csv)

## How to apply

Run eRah's five-step pipeline (noise filtering/baseline removal, deconvolution via CMLC/OSD with deconvolveComp(), alignment with alignComp(), missing compound recovery with recMissComp(), and identification with identifyComp() against a MSP library such as the bundled MassBank library, GMD, or NIST-exported MSP spectra) to obtain the aligned compound table via dataList(ex). Load the supporting eRah identification file (e.g. Supporting File 4 from the article) and subset rows for the metabolites of interest — here lactic acid, leucine 1TMS/2TMS, serine, 5-oxoproline, glutamic acid, ornithine, and lysine. Split relative concentrations by class label (11 HIAE patients vs 14 CTR controls), compute percent variation as 100*(mean(HIAE)-mean(CTR))/mean(CTR), and in R compute two-sided Wilcoxon-Mann-Whitney rank-sum p-values per metabolite. Assemble metabolite name, percent variation, and p-value into a table (e.g. table3_metabolite_stats.csv). Validation: confirm all recomputed p-values fall below the significance threshold reported in the paper (p<0.01) and that percent-variation direction/magnitude matches Table 3.

## Related tools

- **erah** (R package that performs deconvolution, alignment, missing compound recovery and library-based identification, outputting the compound name/score/area table consumed by this skill.) — http://CRAN.R-project.org/package=erah
- **erah-devel** (Development version of eRah installed from GitHub when CRAN release is insufficient; documentation and issue tracker for bugs.) — https://github.com/xdomingoal/erah-devel
- **XCMS** (Reference peak-picking algorithm (centWave) used in the article for comparison against eRah's quantitative results; not part of the eRah pipeline itself.)
- **MetAlign** (Third-party peak-picking tool used in the article for benchmarking eRah's quantification; not part of the eRah pipeline.)
- **ChromaTOF** (Commercial software (LECO Corp.) used in the article as a benchmark for eRah's quantitative results.)

## Examples

```
install.packages('erah'); ex <- deconvolveComp(...); ex <- alignComp(...); ex <- recMissComp(...); ex <- identifyComp(...); data <- dataList(ex)
```

## Evaluation signals

- All recomputed p-values for the eight dysregulated metabolites fall below 0.01 (matching Table 3 of the paper).
- Percent-of-variation values for lactic acid, leucine, serine, 5-oxoproline, glutamic acid, ornithine and lysine are positive (elevated in HIAE) and of comparable magnitude to Table 3.
- Identification output from identifyComp()/idList() shows matching scores >80.0 for compounds matched to GMD/MassBank reference spectra.
- The final dataList(ex) table contains one row per aligned compound with non-missing integrated area for each serum sample.
- After recMissComp(), compounds are re-identified (re-running identifyComp) as the article explicitly instructs.

## Limitations

- No explicit version number for eRah is stated in the article; reproducibility requires noting the eRah/erah-devel version used.
- NIST library spectra are only readable with NIST software — eRah requires MSP-format libraries, so NIST libraries must be exported first.
- Validation scope in the article was limited to GC–TOF MS serum samples; performance on other instruments or matrices is untested here.
- The HIAE-vs-control comparative methods are deferred to supporting methods rather than the main text, so exact processing parameters for that dataset are only available via Supporting Files.

## Evidence

- [abstract] erah outputs a table with compound names, matching scores and integrated area of compounds for each sample: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [methods] The compounds in data are deconvolved using `deconvolveComp()`: "The compounds in data are deconvolved using `deconvolveComp()`"
- [methods] we always have to re-identify the compounds after executing the missing compound recovery step: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [results] These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank.: "These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank."
- [results] Note that NIST library can only be read with NIST software. However, e Rah has a function to export all the spectra found in a given experiment to MSP format.: "Note that NIST library can only be read with NIST software. However, e Rah has a function to export all the spectra found in a given experiment to MSP format."
