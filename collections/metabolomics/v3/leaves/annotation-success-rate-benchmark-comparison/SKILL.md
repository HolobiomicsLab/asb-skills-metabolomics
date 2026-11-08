---
name: annotation-success-rate-benchmark-comparison
description: Use when you have deconvolved MS2 spectra from SWATH-MS untargeted metabolomics
  data and need to quantify what fraction can be assigned a metabolite/formula annotation
  by MetDNA or Sirius, compared against a reported ~90% benchmark.
when_to_use_negative:
- Input spectra are raw multiplexed SWATH-MS MS/MS spectra that still need deconvolution
  — use DecoMetDIA's deconvolution skill first rather than benchmarking annotation
  directly
- Input is already a metabolite feature/annotation table with no MS2 spectra to annotate
- The goal is accuracy comparison against experimental DDA MS2 spectra rather than
  downstream annotation success-rate estimation
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DecoMetDIA
  role: R package used to deconvolve multiplexed SWATH-MS MS/MS spectra into metabolite
    MS2 spectra that are then submitted for annotation
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: MetDNA
  role: Downstream annotation tool applied to the deconvolved spectra using its metabolite-reaction-network
    propagation workflow
- name: Sirius
  role: Downstream annotation tool used for formula-based annotation of the deconvolved
    precursor/fragment peaks
- name: R
  role: Language in which DecoMetDIA is implemented (R package with Rcpp code) and
    in which the benchmark workflow is driven
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:99fefa558a1cf3be05bacff393c80fcab7df826f2c80d903d6b70e3795f7f6e1
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
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# annotation-success-rate-benchmark-comparison

## Summary

Benchmarks the annotation success rate of deconvolved MS2 spectra (e.g. from DecoMetDIA SWATH-MS processing) by submitting them to metabolite annotation tools such as MetDNA and Sirius, then compares the achieved fraction against the paper's reported ~90% success rate across biological samples.

## When to use

Use when you have deconvolved MS2 spectra from SWATH-MS untargeted metabolomics data and need to quantify what fraction can be assigned a metabolite/formula annotation by MetDNA or Sirius, compared against a reported ~90% benchmark.

## When NOT to use

- Input spectra are raw multiplexed SWATH-MS MS/MS spectra that still need deconvolution — use DecoMetDIA's deconvolution skill first rather than benchmarking annotation directly
- Input is already a metabolite feature/annotation table with no MS2 spectra to annotate
- The goal is accuracy comparison against experimental DDA MS2 spectra rather than downstream annotation success-rate estimation

## Inputs

- Deconvolved MS2 spectra produced by DecoMetDIA from biological-sample SWATH-MS data (spectral output enabled via is.output.spec.smp)
- Precursor m/z and fragment peak lists for each deconvolved spectrum

## Outputs

- annotation_success_rate.csv with a success-rate field for every deconvolved spectrum
- Per-spectrum annotation assignments from MetDNA (metabolite) and Sirius (formula)
- Overall annotation success rate (fraction) comparable to the ~90% benchmark

## How to apply

First obtain deconvolved MS2 spectra by running the exported DecoMetDIA() function on biological-sample SWATH-MS data with spectral output enabled (is.output.spec.smp). Submit the deconvolved spectra to MetDNA, which uses its default metabolite-reaction-network propagation workflow for annotation, and in parallel submit the same spectra to Sirius for formula-based annotation of the deconvolved precursor/fragment peaks. Compute the annotation success rate as the fraction of deconvolved MS2 spectra assigned a metabolite or formula annotation by MetDNA or Sirius. Tabulate per-spectrum annotations and the overall rate, then qualitatively compare against the reported ~90% success rate for deconvolved MS2 spectra in various biological samples.

## Related tools

- **DecoMetDIA** (R package used to deconvolve multiplexed SWATH-MS MS/MS spectra into metabolite MS2 spectra that are then submitted for annotation) — https://github.com/ZhuMSLab/DecoMetDIA
- **MetDNA** (Downstream annotation tool applied to the deconvolved spectra using its metabolite-reaction-network propagation workflow)
- **Sirius** (Downstream annotation tool used for formula-based annotation of the deconvolved precursor/fragment peaks)
- **R** (Language in which DecoMetDIA is implemented (R package with Rcpp code) and in which the benchmark workflow is driven)

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA")
```

## Evaluation signals

- annotation_success_rate.csv exists and contains a success-rate field for every deconvolved spectrum
- The overall annotation success rate is qualitatively consistent with the ~90% rate reported for deconvolved MS2 spectra across biological samples
- Every deconvolved spectrum has a per-spectrum annotation record from at least one of MetDNA or Sirius
- The denominator of the rate equals the number of deconvolved MS2 spectra exported by DecoMetDIA with is.output.spec.smp enabled

## Limitations

- The ~90% figure is a qualitative consistency target, not an exact reproduction threshold; rates will vary with sample type and tool versions
- Success is scored as annotation by MetDNA or Sirius; a spectrum failing both tools may reflect tool-specific coverage limits rather than failed deconvolution
- DecoMetDIA requires compilation (Rcpp); on Windows OS, RTools must be installed first before installing the package

## Evidence

- [abstract] ~90% annotation benchmark: "about 90% of deconvolved MS2 spectra in various biological samples were successfully annotated using software tools such as MetDNA and Sirius"
- [methods] spectral output from DecoMetDIA: "DecoMetDIA was developed as an R package with Rcpp code."
- [readme] installation procedure: "The install can be done with ```install_github()```  method in devtools"
