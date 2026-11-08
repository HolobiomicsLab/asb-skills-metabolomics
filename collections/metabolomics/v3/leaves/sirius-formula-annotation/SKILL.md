---
name: sirius-formula-annotation
description: Use when you have deconvolved (or DDA) MS2 spectra from SWATH-MS untargeted
  metabolomics and need to assign molecular formulas to precursor/fragment peaks,
  or when benchmarking annotation success rate of DecoMetDIA output.
when_to_use_negative:
- The input spectra are not deconvolved multiplexed SWATH-MS MS/MS spectra (e.g.,
  already-annotated feature tables or raw MS1-only files) — DecoMetDIA deconvolution
  with is.ms1.filter and int.filter steps must come first
- The goal is comparison of deconvolution performance itself rather than downstream
  annotation — the paper uses MS-DIAL for that comparison
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Sirius
  role: Formula-based annotation of deconvolved precursor/fragment peaks from DecoMetDIA
    spectra
- name: MetDNA
  role: Parallel annotation tool using its metabolite-reaction-network propagation
    workflow; used with Sirius to reach ~90% annotation of deconvolved spectra
- name: DecoMetDIA
  role: Produces the deconvolved MS2 spectra submitted to Sirius; R package with Rcpp
    code run via the exported DecoMetDIA() function
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: R
  role: Language in which DecoMetDIA (and the surrounding workflow scripts) is implemented
    and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:2a8d6f8b02faab27373b105f94acd4db369e0c39ccc6de46268d604d5848e242
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZhuMSLab/DecoMetDIA
  tools:
  - DecoMetDIA
  - MetDNA
  - R
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sirius-formula-annotation

## Summary

Formula-based annotation of deconvolved MS2 spectra (e.g., spectra reconstructed by DecoMetDIA from SWATH-MS data) using Sirius, as a downstream metabolite-annotation step benchmarked in this paper against MetDNA and together credited with annotating about 90% of deconvolved spectra from biological samples.

## When to use

Use when you have deconvolved (or DDA) MS2 spectra from SWATH-MS untargeted metabolomics and need to assign molecular formulas to precursor/fragment peaks, or when benchmarking annotation success rate of DecoMetDIA output.

## When NOT to use

- The input spectra are not deconvolved multiplexed SWATH-MS MS/MS spectra (e.g., already-annotated feature tables or raw MS1-only files) — DecoMetDIA deconvolution with is.ms1.filter and int.filter steps must come first
- The goal is comparison of deconvolution performance itself rather than downstream annotation — the paper uses MS-DIAL for that comparison

## Inputs

- Deconvolved MS2 spectra produced by DecoMetDIA from SWATH-MS sample data (spectral output via is.output.spec.smp)
- Precursor/fragment peak lists (m/z and intensity) for each deconvolved spectrum
- Optional: experimental DDA MS2 spectra for accuracy comparison

## Outputs

- Per-spectrum formula/metabolite annotations assigned by Sirius
- Annotation success-rate table (e.g., annotation_success_rate.csv) with a success-rate field for every deconvolved spectrum
- Overall annotation success rate comparable to the reported ~90%

## How to apply

First obtain deconvolved MS2 spectra by running DecoMetDIA's exported DecoMetDIA() function on SWATH-MS sample data with spectral output enabled (is.output.spec.smp); DecoMetDIA models coeluted fragment-ion chromatographic peaks as a linear combination of multiple model peaks to reconstruct each metabolite's MS2 spectrum. In parallel with MetDNA, submit these deconvolved spectra to Sirius for formula-based annotation of the deconvolved precursor/fragment peaks. Compute the annotation success rate as the fraction of deconvolved MS2 spectra assigned a formula/metabolite annotation, tabulate per-spectrum results (e.g., annotation_success_rate.csv), and compare against the ~90% rate reported in the paper for various biological samples. Accuracy of the reconstructed spectra can additionally be cross-checked against experimental DDA MS2 spectra.

## Related tools

- **Sirius** (Formula-based annotation of deconvolved precursor/fragment peaks from DecoMetDIA spectra)
- **MetDNA** (Parallel annotation tool using its metabolite-reaction-network propagation workflow; used with Sirius to reach ~90% annotation of deconvolved spectra)
- **DecoMetDIA** (Produces the deconvolved MS2 spectra submitted to Sirius; R package with Rcpp code run via the exported DecoMetDIA() function) — https://github.com/ZhuMSLab/DecoMetDIA
- **R** (Language in which DecoMetDIA (and the surrounding workflow scripts) is implemented and executed)

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA")
```

## Evaluation signals

- Output file (annotation_success_rate.csv) exists and contains a success-rate field for every deconvolved spectrum
- Achieved annotation success rate is qualitatively consistent with the reported ~90% of deconvolved MS2 spectra annotated via MetDNA and Sirius
- Deconvolved spectra show high accuracy when compared against experimental DDA MS2 spectra
- Annotations cover metabolites across a variety of biological samples with high coverage
- Per-spectrum annotation table is complete (each deconvolved spectrum tabulated as annotated or not)

## Limitations

- The ~90% success rate applies to deconvolved MS2 spectra from various biological samples; spectra of poor deconvolution quality may not be annotatable
- DecoMetDIA must be compiled during installation (Rcpp code); on Windows, RTools must be installed first
- The paper's comparison to MS-DIAL indicates the deconvolution feeding Sirius performs best for small polar metabolites

## Evidence

- [methods] DecoMetDIA workflow: run DecoMetDIA() with spectral output on sample SWATH-MS data: [evidence span withheld — anchor status: not_found]
