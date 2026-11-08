---
name: linear-combination-decomposition-of-fragment-ion-peaks
description: Use when SWATH-MS-based untargeted metabolomics data contain coeluted,
  unresolved chromatographic peaks of fragment ions in multiplexed MS/MS spectra that
  must be separated into metabolite-specific MS2 spectra.
when_to_use_negative:
- Input MS2 spectra are already resolved by data-dependent acquisition (DDA), so no
  multiplexed deconvolution is needed — direct DDA spectra comparison suffices.
- The task is annotating already-deconvolved MS2 spectra, which is the downstream
  job of MetDNA or Sirius rather than this skill.
- Input is non-SWATH metabolomics data outside the scope of tools developed 'to process
  SWATH-MS based data for metabolomics'.
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: DecoMetDIA
  role: Reference R package implementing model-peak selection and linear-combination
    decomposition of multiplexed SWATH MS/MS spectra
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: R
  role: Runtime environment in which the deconvolution pipeline (R package) is executed
- name: Rcpp
  role: C++ integration layer used to compile the DecoMetDIA package code
- name: devtools
  role: Provides install_github() to install the reference implementation from ZhuMSLab/DecoMetDIA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:b8c8a3abf40c7f53dda88f94210fee47d4ef7a4c4d0b283e6de2b0f212b687b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZhuMSLab/DecoMetDIA
  tools:
  - DecoMetDIA
  - R
  - Rcpp
  - devtools
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# linear-combination-decomposition-of-fragment-ion-peaks

## Summary

Deconvolves multiplexed SWATH-MS MS/MS spectra by selecting multiple model chromatographic peaks that model coeluted, unresolved fragment-ion peaks, then decomposing each multiplexed spectrum as a linear combination of those model peaks to recover metabolite-specific MS2 spectra.

## When to use

Use when SWATH-MS-based untargeted metabolomics data contain coeluted, unresolved chromatographic peaks of fragment ions in multiplexed MS/MS spectra that must be separated into metabolite-specific MS2 spectra.

## When NOT to use

- Input MS2 spectra are already resolved by data-dependent acquisition (DDA), so no multiplexed deconvolution is needed — direct DDA spectra comparison suffices.
- The task is annotating already-deconvolved MS2 spectra, which is the downstream job of MetDNA or Sirius rather than this skill.
- Input is non-SWATH metabolomics data outside the scope of tools developed 'to process SWATH-MS based data for metabolomics'.

## Inputs

- Raw SWATH-MS MS1 and MS2 spectra with SWATH window setup (multiplexed, coeluted fragment-ion peaks)
- Bundled sample/test SWATH input spectra from the ZhuMSLab/DecoMetDIA package

## Outputs

- Deconvolved, metabolite-specific MS2 spectra (e.g. deconvolved_ms2_spectra.mgf)
- Selected model chromatographic peaks used in the linear-combination decomposition

## How to apply

Install the reference DecoMetDIA R package (Rcpp-compiled; on Windows install RTools first) via devtools::install_github("ZhuMSLab/DecoMetDIA"). Load raw MS1/MS2 SWATH spectra together with the SWATH window setup into R. Implement the model-peak selection stage: select multiple model chromatographic peaks to model the coeluted and unresolved fragment-ion peaks in each multiplexed spectrum. Implement the decomposition stage: express each multiplexed MS/MS spectrum as a linear combination of the selected model peaks, yielding deconvolved MS2 spectra per metabolite. Apply default MS1/MS2 quality settings (is.ms1.filter, int.filter) and export the result, e.g. deconvolved_ms2_spectra.mgf. Validate by cross-checking against the reference DecoMetDIA() output and comparing deconvolved spectra to experimental DDA MS2 spectra.

## Related tools

- **DecoMetDIA** (Reference R package implementing model-peak selection and linear-combination decomposition of multiplexed SWATH MS/MS spectra) — https://github.com/ZhuMSLab/DecoMetDIA
- **R** (Runtime environment in which the deconvolution pipeline (R package) is executed)
- **Rcpp** (C++ integration layer used to compile the DecoMetDIA package code)
- **devtools** (Provides install_github() to install the reference implementation from ZhuMSLab/DecoMetDIA)

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA")
```

## Evaluation signals

- Re-implementation output matches the reference DecoMetDIA() function run on the same sample input with is.ms1.filter and int.filter at defaults.
- Deconvolved MS2 spectra show high accuracy when compared against experimental DDA MS2 spectra.
- Exported deconvolved_ms2_spectra.mgf is well-formed and covers metabolites from the SWATH-MS samples.
- Roughly ~90% of deconvolved MS2 spectra are successfully annotated downstream via MetDNA or Sirius.
- For small polar metabolites, performance is comparable to or better than MS-DIAL's deconvolution.

## Limitations

- Compilation of the Rcpp code is required at install time; on Windows OS, RTools must be installed first.
- The package is distributed under CC BY-NC-ND 4.0 (non-commercial, no derivatives).
- Validation coverage is anchored on comparison to experimental DDA MS2 spectra; no discussion-section caveats or changelog were available in the source document.

## Evidence
