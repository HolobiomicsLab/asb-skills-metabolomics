---
name: deconvolved-spectra-quality-assessment
description: Use when you have deconvolved MS2 spectra produced by DecoMetDIA (or
  another SWATH-MS deconvolution tool) from biological samples and need to quantify
  how many spectra are annotatable, or to benchmark deconvolution quality against
  the reported ~90% annotation success rate.
when_to_use_negative:
- Input spectra are raw multiplexed (undeconvolved) SWATH-MS MS/MS spectra — run DecoMetDIA's
  deconvolution first rather than annotating directly
- Input is an already-annotated feature or metabolite-identification table — no annotation
  step is needed
- The comparison target is deconvolution accuracy itself (spectral similarity to DDA
  MS2) rather than downstream annotatability
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DecoMetDIA
  role: Produces the deconvolved MS2 spectra being assessed; run with spectral output
    enabled (is.output.spec.smp) on biological-sample SWATH-MS data
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: MetDNA
  role: Downstream metabolite annotation tool applied to the deconvolved spectra to
    assign metabolite identities via its metabolite-reaction-network propagation workflow
- name: Sirius
  role: Downstream formula-based annotation tool applied in parallel to the deconvolved
    precursor/fragment peaks
- name: R
  role: Language of the DecoMetDIA package (with Rcpp code) used to run the deconvolution
    and export spectra for annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:150a27c2a48e876cb9dd95b62fe9e6f797c9de4825214f89321acc4b79574790
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZhuMSLab/DecoMetDIA
  tools:
  - DecoMetDIA
  - MetDNA
  - R
  - Sirius
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# deconvolved-spectra-quality-assessment

## Summary

Assesses the quality of DecoMetDIA-deconvolved MS2 spectra from SWATH-MS untargeted metabolomics by submitting them to downstream metabolite annotation tools (MetDNA and Sirius) and computing the fraction of spectra that receive a metabolite or formula annotation, benchmarked against the paper's reported ~90% success rate.

## When to use

Use when you have deconvolved MS2 spectra produced by DecoMetDIA (or another SWATH-MS deconvolution tool) from biological samples and need to quantify how many spectra are annotatable, or to benchmark deconvolution quality against the reported ~90% annotation success rate.

## When NOT to use

- Input spectra are raw multiplexed (undeconvolved) SWATH-MS MS/MS spectra — run DecoMetDIA's deconvolution first rather than annotating directly
- Input is an already-annotated feature or metabolite-identification table — no annotation step is needed
- The comparison target is deconvolution accuracy itself (spectral similarity to DDA MS2) rather than downstream annotatability

## Inputs

- Deconvolved MS2 spectra exported from DecoMetDIA (spectral output enabled via is.output.spec.smp) from biological-sample SWATH-MS runs
- Precursor m/z and retention-time information accompanying each deconvolved MS2 spectrum
- MetDNA and Sirius accounts/installs for metabolite and formula-based annotation

## Outputs

- Per-spectrum annotation table (e.g., annotation_success_rate.csv) with an annotation/success-rate field for every deconvolved spectrum
- Overall annotation success rate (fraction of deconvolved MS2 spectra annotated by MetDNA or Sirius), expected to be qualitatively consistent with ~90%

## How to apply

First obtain deconvolved MS2 spectra by running DecoMetDIA's exported function on SWATH-MS data from biological samples with spectral output enabled (is.output.spec.smp). Then submit the deconvolved spectra to MetDNA, which uses its metabolite-reaction-network propagation workflow, and in parallel to Sirius, which performs formula-based annotation of precursor and fragment peaks. Compute the annotation success rate as the fraction of deconvolved MS2 spectra assigned a metabolite or formula annotation by either tool. Tabulate per-spectrum annotations and the overall success rate, then compare with the paper's reported ~90% success rate for deconvolved spectra across various biological samples. Write the per-spectrum results and the overall rate to a table (e.g., annotation_success_rate.csv).

## Related tools

- **DecoMetDIA** (Produces the deconvolved MS2 spectra being assessed; run with spectral output enabled (is.output.spec.smp) on biological-sample SWATH-MS data) — https://github.com/ZhuMSLab/DecoMetDIA
- **MetDNA** (Downstream metabolite annotation tool applied to the deconvolved spectra to assign metabolite identities via its metabolite-reaction-network propagation workflow)
- **Sirius** (Downstream formula-based annotation tool applied in parallel to the deconvolved precursor/fragment peaks)
- **R** (Language of the DecoMetDIA package (with Rcpp code) used to run the deconvolution and export spectra for annotation)

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA"); DecoMetDIA(..., is.output.spec.smp = TRUE)
```

## Evaluation signals

- The output file (e.g., annotation_success_rate.csv) exists and contains a success-rate field for every deconvolved spectrum
- The achieved annotation success rate is qualitatively consistent with the ~90% rate the paper reports for deconvolved MS2 spectra in various biological samples
- Per-spectrum annotations are tabulated for both MetDNA and Sirius submissions, allowing each spectrum's annotation source to be traced
- The fraction annotated by either tool (MetDNA or Sirius) is computed over the same set of deconvolved spectra, not just a subset

## Limitations

- The ~90% annotation success rate is reported for deconvolved spectra from various biological samples; different sample types, instruments, or parameter settings may yield different rates and only qualitative consistency is claimed
- This skill measures downstream annotatability, not spectral accuracy — the paper separately assessed accuracy by comparison to experimental DDA MS2 spectra
- MetDNA and Sirius are 'such as' examples of annotation tools; results depend on the specific annotation software and its version/configuration

## Evidence

- [abstract] About 90% of deconvolved MS2 spectra in various biological samples were successfully annotated using tools such as MetDNA and Sirius: "about 90% of deconvolved MS2 spectra in various biological samples were successfully annotated using software tools such as MetDNA and Sirius"
- [abstract] MetDNA and Sirius are downstream annotation tools for deconvolved spectra: "annotated using software tools such as MetDNA and Sirius"
- [methods] DecoMetDIA is an R package with Rcpp code and needs installation: "DecoMetDIA was developed as an R package with Rcpp code."
- [readme] Install DecoMetDIA from GitHub via devtools before running the deconvolution and spectral export: "devtools::install_github("ZhuMSLab/DecoMetDIA")"
