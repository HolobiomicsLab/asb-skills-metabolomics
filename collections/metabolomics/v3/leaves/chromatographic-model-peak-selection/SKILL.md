---
name: chromatographic-model-peak-selection
description: Use when SWATH-MS data have multiplexed MS/MS spectra in which fragment
  ions co-elute and chromatographic peaks are unresolved, and metabolite-specific
  MS2 spectra must be reconstructed for downstream annotation.
when_to_use_negative:
- Input data are already deconvolved or come from data-dependent acquisition (DDA)
  where MS/MS spectra are not multiplexed.
- The analysis goal is general feature detection or peak picking on MS1 data rather
  than deconvolution of multiplexed MS/MS spectra in SWATH-MS-based untargeted metabolomics.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DecoMetDIA
  role: R package (with Rcpp code) that implements the model-peak selection and linear-combination
    decomposition of multiplexed SWATH MS/MS spectra
  repo: https://github.com/ZhuMSLab/DecoMetDIA
- name: devtools
  role: Provides install_github() to install the DecoMetDIA package from GitHub
- name: Rcpp
  role: C++ integration used for the compiled code inside the DecoMetDIA R package
- name: MetDNA
  role: Used downstream to annotate the deconvolved MS2 spectra
- name: Sirius
  role: Used downstream to annotate the deconvolved MS2 spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02655
    title: 'DecoMetDIA: Deconvolution of Multiplexed MS/MS Spectra for Metabolite
      Identification in SWATH-MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:1e84cf5548fc71766b95317de75618e5ac6269e9421deed31dcc68701bfa8ea7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZhuMSLab/DecoMetDIA
  tools:
  - DecoMetDIA
  - MetDNA
  - R
  - Rcpp
  - Sirius
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# Chromatographic model-peak selection

## Summary

DecoMetDIA skill for selecting multiple model chromatographic peaks that represent co-eluted and unresolved fragment-ion peaks in multiplexed SWATH-MS spectra, so that each multiplexed MS/MS spectrum can be decomposed into a linear combination of model peaks to recover metabolite-specific MS2 spectra.

## When to use

Use when SWATH-MS data have multiplexed MS/MS spectra in which fragment ions co-elute and chromatographic peaks are unresolved, and metabolite-specific MS2 spectra must be reconstructed for downstream annotation.

## When NOT to use

- Input data are already deconvolved or come from data-dependent acquisition (DDA) where MS/MS spectra are not multiplexed.
- The analysis goal is general feature detection or peak picking on MS1 data rather than deconvolution of multiplexed MS/MS spectra in SWATH-MS-based untargeted metabolomics.

## Inputs

- Raw SWATH-MS MS1 chromatographic data with SWATH window setup
- Multiplexed SWATH MS/MS spectra containing co-eluted, unresolved fragment-ion chromatographic peaks

## Outputs

- Selected model chromatographic peaks representing co-eluted fragment-ion peaks
- Deconvolved, metabolite-specific MS2 spectra (exported as deconvolved_ms2_spectra.mgf)

## How to apply

Run DecoMetDIA from ZhuMSLab/DecoMetDIA, an R package with Rcpp code that must be compiled (on Windows install RTools first) via devtools::install_github("ZhuMSLab/DecoMetDIA"). Load raw MS1/MS2 SWATH data plus the SWATH window setup, then execute the model-peak-selection stage: DecoMetDIA selects multiple model peaks to model the co-eluted and unresolved chromatographic peaks of fragment ions in multiplexed spectra. Next, the decomposition stage expresses each multiplexed MS/MS spectrum as a linear combination of the selected model peaks, yielding deconvolved MS2 spectra per metabolite, exported as deconvolved_ms2_spectra.mgf. Settings such as is.ms1.filter and int.filter (with their companion parameters thr.ms1.filter, filed.ms1.filter, ppm.ms2.mtp, minfrac.vote) are controllable filter parameters; keep them at defaults when cross-checking against the reference DecoMetDIA() function. Assess the deconvolved MS2 spectra by comparing them to experimental data-dependent acquisition (DDA) MS2 spectra, mirroring the paper's reported high accuracy and roughly 90% annotation coverage.

## Related tools

- **DecoMetDIA** (R package (with Rcpp code) that implements the model-peak selection and linear-combination decomposition of multiplexed SWATH MS/MS spectra) — https://github.com/ZhuMSLab/DecoMetDIA
- **devtools** (Provides install_github() to install the DecoMetDIA package from GitHub)
- **Rcpp** (C++ integration used for the compiled code inside the DecoMetDIA R package)
- **MetDNA** (Used downstream to annotate the deconvolved MS2 spectra)
- **Sirius** (Used downstream to annotate the deconvolved MS2 spectra)

## Examples

```
devtools::install_github("ZhuMSLab/DecoMetDIA")
```

## Evaluation signals

- Cross-check the re-implementation against the reference DecoMetDIA() function on the same sample input with is.ms1.filter and int.filter at defaults, confirming consistent deconvolution behavior.
- Compare the deconvolved MS2 spectra against experimental DDA MS2 spectra; the paper reports high accuracy of the deconvolved spectra by this comparison.
- Check that roughly 90% of the deconvolved MS2 spectra in various biological samples can be successfully annotated with tools such as MetDNA and Sirius.
- Verify output is a valid MGF file (deconvolved_ms2_spectra.mgf) containing metabolite-specific spectra rather than multiplexed mixtures.
- Compare performance on small polar metabolites against MS-DIAL, as the paper demonstrates DecoMetDIA performs well for these.

## Limitations

- DecoMetDIA is targeted at SWATH-MS-based untargeted metabolomics; the article frames it around multiplexed MS/MS spectra of that acquisition mode.
- Installing the package requires compiling Rcpp code; on Windows, RTools must be installed first, which can block setup without the proper toolchain.
- The README does not include a changelog or detailed parameter documentation beyond the install instructions, so parameter tuning relies on the function defaults and the publication.

## Evidence

- [abstract] multiple model peaks are selected to model the coeluted and unresolved chromatographic peaks of fragment ions in multiplexed spectra and decompose them into a linear combination of the model peaks: "multiple model peaks are selected to model the coeluted and unresolved chromatographic peaks of fragment ions in multiplexed spectra and decompose them into a linear combination of the model peaks"
- [abstract] the deconvolved MS2 spectra from DecoMetDIA were of high accuracy through comparison to the experimental MS2 spectra from data-dependent acquisition (DDA): "the deconvolved MS2 spectra from DecoMetDIA were of high accuracy through comparison to the experimental MS2 spectra from data-dependent acquisition (DDA)"
- [abstract] about 90% of deconvolved MS2 spectra in various biological samples were successfully annotated using software tools such as MetDNA and Sirius: "about 90% of deconvolved MS2 spectra in various biological samples were successfully annotated using software tools such as MetDNA and Sirius"
- [methods] DecoMetDIA was developed as an R package with Rcpp code.: "DecoMetDIA was developed as an R package with Rcpp code."
- [methods] The install can be done with ```install_github()``` method in devtools as following: "The install can be done with ```install_github()``` method in devtools as following"
- [methods] On Windows OS, RTools should be installed first.: "On Windows OS, RTools should be installed first."
- [abstract] The comparison of DecoMetDIA to other deconvolution software such as MS-DIAL demonstrated that it performs very well for small polar metabolites: "The comparison of DecoMetDIA to other deconvolution software such as MS-DIAL demonstrated that it performs very well for small polar metabolites"
