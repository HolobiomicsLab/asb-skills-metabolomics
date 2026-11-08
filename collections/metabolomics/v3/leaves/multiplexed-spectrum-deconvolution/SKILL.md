---
name: multiplexed-spectrum-deconvolution
description: Use when SWATH-MS-based untargeted metabolomics yields multiplexed MS/MS
  spectra with co-eluted, unresolved fragment-ion chromatographic peaks that must
  be deconvolved into metabolite-specific MS2 spectra.
when_to_use_negative:
- Input MS/MS spectra are already metabolite-specific DDA spectra, not multiplexed
  SWATH spectra — no co-eluted fragment-ion peaks need deconvolving
- The goal is generic DIA-MS processing without metabolite identification focus; the
  paper benchmarks DecoMetDIA specifically for small polar metabolites in untargeted
  metabolomics
- Only feature/intensity tables are available without raw chromatographic peak shapes,
  since the method models co-eluted chromatographic peaks of fragment ions
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DecoMetDIA
  role: Reference R package (with Rcpp code) implementing model-peak selection and
    linear-combination decomposition of multiplexed SWATH-MS/MS spectra
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: R
  role: Language in which DecoMetDIA is developed and executed
- name: Rcpp
  role: Compiled C++ backend of the DecoMetDIA package; compilation is needed during
    install
- name: devtools
  role: Provides install_github() to install DecoMetDIA from GitHub
- name: RTools
  role: Toolchain that must be installed on Windows before compiling the Rcpp code
- name: MetDNA
  role: Downstream annotation tool applied to deconvolved MS2 spectra (used for validation
    coverage)
- name: Sirius
  role: Downstream annotation tool applied to deconvolved MS2 spectra (used for validation
    coverage)
- name: MS-DIAL
  role: Comparison/benchmark deconvolution software; DecoMetDIA was shown to perform
    very well for small polar metabolites relative to it
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:db44bcc2637074e9316f7253b183fb27264dc23f88777c3625c5d2138aeba881
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZhuMSLab/DecoMetDIA
  tools:
  - DecoMetDIA
  - MS-DIAL
  - MetDNA
  - R
  - RTools
  - Rcpp
  - Sirius
  - devtools
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# multiplexed-spectrum-deconvolution

## Summary

Deconvolves multiplexed SWATH-MS MS/MS spectra into metabolite-specific MS2 spectra by selecting multiple model chromatographic peaks and decomposing co-eluted fragment-ion peaks as their linear combination, as implemented in the DecoMetDIA R package.

## When to use

Use when SWATH-MS-based untargeted metabolomics yields multiplexed MS/MS spectra with co-eluted, unresolved fragment-ion chromatographic peaks that must be deconvolved into metabolite-specific MS2 spectra.

## When NOT to use

- Input MS/MS spectra are already metabolite-specific DDA spectra, not multiplexed SWATH spectra — no co-eluted fragment-ion peaks need deconvolving
- The goal is generic DIA-MS processing without metabolite identification focus; the paper benchmarks DecoMetDIA specifically for small polar metabolites in untargeted metabolomics
- Only feature/intensity tables are available without raw chromatographic peak shapes, since the method models co-eluted chromatographic peaks of fragment ions

## Inputs

- Raw MS1/MS2 SWATH-MS spectra from untargeted metabolomics runs
- SWATH window setup (isolation window definitions for the multiplexed acquisition)

## Outputs

- Deconvolved, metabolite-specific MS2 spectra (exportable as deconvolved_ms2_spectra.mgf)
- MS2 spectra suitable for downstream annotation with MetDNA or Sirius

## How to apply

Install the reference DecoMetDIA R package (Rcpp code; on Windows install RTools first) via devtools::install_github('ZhuMSLab/DecoMetDIA'). Load raw MS1/MS2 SWATH input spectra together with the SWATH window setup, then run the two-stage algorithm: (1) model-peak selection, where multiple model chromatographic peaks are chosen to model the co-eluted and unresolved chromatographic peaks of fragment ions; (2) decomposition, where each multiplexed MS/MS spectrum is expressed as a linear combination of the selected model peaks to yield deconvolved MS2 spectra. Keep filtering parameters such as is.ms1.filter, thr.ms1.filter, filed.ms1.filter, ppm.ms2.mtp, int.filter, and minfrac.vote at their defaults for a baseline run. Export the deconvolved spectra (e.g., MGF) and validate by comparison to experimental DDA MS2 spectra, mirroring the paper's acceptance criterion of high-accuracy reconstruction and ~90% annotation coverage via MetDNA/Sirius.

## Related tools

- **DecoMetDIA** (Reference R package (with Rcpp code) implementing model-peak selection and linear-combination decomposition of multiplexed SWATH-MS/MS spectra) — https://github.com/ZhuMSLab/DecoMetDIA
- **R** (Language in which DecoMetDIA is developed and executed)
- **Rcpp** (Compiled C++ backend of the DecoMetDIA package; compilation is needed during install)
- **devtools** (Provides install_github() to install DecoMetDIA from GitHub)
- **RTools** (Toolchain that must be installed on Windows before compiling the Rcpp code)
- **MetDNA** (Downstream annotation tool applied to deconvolved MS2 spectra (used for validation coverage))
- **Sirius** (Downstream annotation tool applied to deconvolved MS2 spectra (used for validation coverage))
- **MS-DIAL** (Comparison/benchmark deconvolution software; DecoMetDIA was shown to perform very well for small polar metabolites relative to it)

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA")
```

## Evaluation signals

- Deconvolved MS2 spectra show high accuracy when compared against experimental DDA MS2 spectra of the same metabolites
- About 90% of deconvolved MS2 spectra across various biological samples are successfully annotated via MetDNA/Sirius
- Re-implementation matches the reference DecoMetDIA() output on the same bundled sample SWATH input spectra (with is.ms1.filter and int.filter at defaults)
- Reconstructed MS2 spectra cover metabolites from a variety of biological samples with high coverage
- Performance on small polar metabolites is at least as good as the MS-DIAL comparison baseline

## Limitations

- Compiling is needed when installing because DecoMetDIA is an R package with Rcpp code; on Windows, RTools must be installed first
- Performance was demonstrated and benchmarked for small polar metabolites; behavior outside that metabolite class is not established by this source
- No discussion-section text or changelog is available in the source document, so failure modes beyond the paper's scope are not documented

## Evidence
