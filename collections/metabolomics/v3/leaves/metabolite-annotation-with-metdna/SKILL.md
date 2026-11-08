---
name: metabolite-annotation-with-metdna
description: Use when you have deconvolved MS2 spectra from SWATH-MS data and need
  to assign metabolite identities via MetDNA's mzCloud-era metabolite-reaction-network
  propagation, or measure annotation success rate.
when_to_use_negative:
- Input spectra are raw multiplexed SWATH MS/MS spectra that still require DecoMetDIA
  deconvolution — annotation should be applied only after spectra are deconvolved
- The goal is deconvolution quality assessment against DDA spectra rather than metabolite
  annotation (use the DecoMetDIA-to-DDA comparison workflow instead)
- Benchmarking deconvolution software performance for small polar metabolites (e.g.,
  against MS-DIAL) rather than downstream annotation
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetDNA
  role: Downstream metabolite annotation of deconvolved MS2 spectra via metabolite-reaction-network
    propagation
- name: Sirius
  role: Formula-based annotation of deconvolved precursor/fragment peaks, used in
    parallel with MetDNA
- name: DecoMetDIA
  role: Upstream deconvolution of multiplexed SWATH-MS/MS spectra to produce the MS2
    spectra that MetDNA annotates; run via exported DecoMetDIA() function with is.output.spec.smp
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: R
  role: Language in which DecoMetDIA is implemented and run as an R package (with
    Rcpp code)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:107800acc4e508a805777f324f748c77dd099de567c1e012d73910296b4ea7f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZhuMSLab/DecoMetDIA
  tools:
  - DecoMetDIA
  - MetDNA
  - R
  - Sirius
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolite-annotation-with-metdna

## Summary

Annotates deconvolved MS2 spectra (e.g., produced by DecoMetDIA from SWATH-MS untargeted metabolomics data) using MetDNA's metabolite-reaction-network propagation workflow, benchmarked against Sirius formula annotation; the paper reports ~90% of deconvolved spectra annotated successfully.

## When to use

Use when you have deconvolved MS2 spectra from SWATH-MS data and need to assign metabolite identities via MetDNA's mzCloud-era metabolite-reaction-network propagation, or measure annotation success rate.

## When NOT to use

- Input spectra are raw multiplexed SWATH MS/MS spectra that still require DecoMetDIA deconvolution — annotation should be applied only after spectra are deconvolved
- The goal is deconvolution quality assessment against DDA spectra rather than metabolite annotation (use the DecoMetDIA-to-DDA comparison workflow instead)
- Benchmarking deconvolution software performance for small polar metabolites (e.g., against MS-DIAL) rather than downstream annotation

## Inputs

- Deconvolved MS2 spectra exported from DecoMetDIA (obtained with is.output.spec.smp spectral output enabled) from biological-sample SWATH-MS acquisitions
- Precursor and fragment m/z values of the deconvolved peaks (for Sirius formula annotation)

## Outputs

- Per-spectrum metabolite/formula annotations from MetDNA and Sirius
- Overall annotation success rate (fraction of deconvolved MS2 spectra annotated), e.g. written to annotation_success_rate.csv

## How to apply

First obtain deconvolved MS2 spectra by running the exported DecoMetDIA() R function on biological-sample SWATH-MS data with spectral output enabled (is.output.spec.smp). Submit the resulting deconvolved spectra to MetDNA, which annotates metabolites via its default metabolite-reaction-network propagation workflow built on the mzCloud-era annotation approach. In parallel, submit the same deconvolved precursor/fragment peaks to Sirius for formula-based annotation as an orthogonal check. Compute the annotation success rate as the fraction of deconvolved MS2 spectra receiving a metabolite/formula annotation, tabulate per-spectrum results, and compare against the paper's reported ~90% success rate across various biological samples. Rationale: DecoMetDIA's model-peak-based deconvolution reconstructs MS2 spectra of sufficient accuracy that downstream annotation tools succeed on the large majority of spectra.

## Related tools

- **MetDNA** (Downstream metabolite annotation of deconvolved MS2 spectra via metabolite-reaction-network propagation)
- **Sirius** (Formula-based annotation of deconvolved precursor/fragment peaks, used in parallel with MetDNA)
- **DecoMetDIA** (Upstream deconvolution of multiplexed SWATH-MS/MS spectra to produce the MS2 spectra that MetDNA annotates; run via exported DecoMetDIA() function with is.output.spec.smp) — https://github.com/ZhuMSLab/DecoMetDIA
- **R** (Language in which DecoMetDIA is implemented and run as an R package (with Rcpp code))

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA")
```

## Evaluation signals

- Output file (e.g., annotation_success_rate.csv) exists and contains a success-rate field for every deconvolved MS2 spectrum
- Achieved annotation success rate is qualitatively consistent with the ~90% rate the paper reports for deconvolved MS2 spectra in various biological samples annotated with MetDNA and Sirius
- Per-spectrum annotation table covers all deconvolved spectra, with each spectrum either assigned a metabolite/formula or explicitly marked unannotated
- MetDNA and Sirius results computed on the same set of deconvolved spectra so rates are directly comparable

## Limitations

- The ~90% annotation success rate is specific to deconvolved spectra from biological samples; lower rates may occur for other sample types
- MetDNA relies on an mzCloud-era metabolite-reaction-network propagation workflow, so annotation coverage depends on that network and library
- DecoMetDIA requires compilation (Rcpp); on Windows OS, RTools must be installed first before installing the package

## Evidence
