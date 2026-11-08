---
name: elastic-net-regularized-regression
description: Use when DDA MS2 spectra are flagged as chimeric (multiple centroids
  inside the isolation window exceeding the acquisition threshold) and their contribution
  to the observed mixed spectrum must be untangled via penalized regression.
when_to_use_negative:
- The MS2 spectrum is not chimeric (single centroid inside the isolation window or
  no intensity above the user-defined acquisition threshold) — no deconvolution is
  needed
- Working on SWATH-DIA data, where PerformDIADeconvolution / reconstructChromPeakSpectra
  relinks precursors with fragment ions instead
- The goal is MS1 feature detection and quantification, handled by the auto-optimized
  LC-MS1 processing module (OptiLCMS) rather than MS2 deconvolution
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalystR 4.0
  role: Implements the self-tuned elastic-net DDA deconvolution via the PerformDDADeconvolution
    function
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the elastic-net regression and deconvolution
    workflow run
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Reads centroided mzML raw data feeding the deconvolution module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:76d1b2e01a07bb53ca819b45700c7ca88a7be270f521fd15e90d17454be44a43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - mzR
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Elastic-net regularized regression

## Summary

MetaboAnalystR 4.0 uses a self-tuned penalized elastic-net regression to deconvolve chimeric DDA MS2 spectra: candidate component spectra are fit against the chimeric 'Spectrum 0' and beta coefficients separate contamination-ion fragments from main-ion fragments.

## When to use

Use when DDA MS2 spectra are flagged as chimeric (multiple centroids inside the isolation window exceeding the acquisition threshold) and their contribution to the observed mixed spectrum must be untangled via penalized regression.

## When NOT to use

- The MS2 spectrum is not chimeric (single centroid inside the isolation window or no intensity above the user-defined acquisition threshold) — no deconvolution is needed
- Working on SWATH-DIA data, where PerformDIADeconvolution / reconstructChromPeakSpectra relinks precursors with fragment ions instead
- The goal is MS1 feature detection and quantification, handled by the auto-optimized LC-MS1 processing module (OptiLCMS) rather than MS2 deconvolution

## Inputs

- Chimeric 'Spectrum 0' (DDA MS2 spectrum with contamination-ion fragments)
- Candidate component spectra (I-IV) from MS1 isotopologue prediction, nearest MS2 scans, spectral library matching, and NetID-rule network prediction
- Isolation-window centroid assignments labeling 'main ion' vs 'contamination ions'

## Outputs

- Normalized deconvoluted MS2 spectrum for the main ion
- Retained original 'Spectrum 0' when no fragment remains or the main ion's beta is 0
- Solution 0: the minimal-residue elastic-net fit across the 110 alpha-lambda permutations

## How to apply

After grouping MS2 spectra by precursor m/z and retention time and flagging chimeric spectra, build candidate component spectra: Candidate Spectrum I (DecoID-style prediction of orphan isotopologues from the MS1 scan), Candidate Spectrum II (clean spectra of the one or two nearest MS2 scans), Candidate Spectrum III (the 'Complete library' spectrum with highest dot-product or spectral-entropy similarity to Spectrum 0), and Candidate Spectrum IV (spectra of unknown ions predicted from abiotic/bio-transformation neighbors via NetID rules on a non-propagatable network, with fragment removal when elemental composition contradicts the unknown ion's formula and a 0-10 similarity penalty). Fit the chimeric 'Spectrum 0' with a penalized elastic-net regression, permuting 110 alpha-lambda combinations (11 alpha values from 0 to 1 in steps of 0.1, 10 lambda values estimated from response-component correlations) and keeping the minimal-residue solution (Solution 0), optimizing alpha and lambda for every individual peak. Use the fitted beta coefficients to remove contamination-ion fragments, normalize the remaining fragments, and export the deconvoluted spectrum for the main ion via the PerformDDADeconvolution routine.

## Related tools

- **MetaboAnalystR 4.0** (Implements the self-tuned elastic-net DDA deconvolution via the PerformDDADeconvolution function) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the elastic-net regression and deconvolution workflow run) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Reads centroided mzML raw data feeding the deconvolution module)

## Examples

```
R: library(MetaboAnalystR); mSet <- InitDataObjects("pktable", "deconv", FALSE); mSet <- PerformDDADeconvolution(mSet)
```

## Evaluation signals

- Every chimeric feature yields either a normalized deconvoluted spectrum or a retained original 'Spectrum 0'
- The output file is parseable and was produced by the PerformDDADeconvolution routine
- The minimal-residue solution (Solution 0) was selected across all 110 alpha-lambda combinations
- Fragments of contamination ions are absent from the exported spectrum; their beta coefficients drove their removal
- For both DDA and DIA datasets, deconvolution should increase the true positive rate of chemical identification by > 40% without increasing false identifications (MetaboAnalystR 4.0 benchmark)

## Limitations

- If no fragment remains after beta-coefficient removal, or the main ion's beta is 0, the original chimeric 'Spectrum 0' is retained unchanged — deconvolution may fail silently on hard cases
- Candidate Spectrum IV relies on NetID transformation-neighbor prediction and removes fragments whose elemental composition contradicts the unknown ion's formula; unknowns without suitable neighbors get a 0-10 similarity penalty and may be poorly modeled
- Reference ('Complete library') spectra are compiled from various instruments and experimental conditions, resulting in potentially varying qualities

## Evidence

- [methods] Elastic-net permutation grid and minimal-residue selection: [evidence span withheld — anchor status: not_found]
- [methods] DDA deconvolution function entry point: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [intro] Chimeric spectra prevalence motivating the regression: "over half of them are chimeric and need to be deconvolved"
- [full_text] Self-tuned regression from the source card finding: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark improvement attributable to deconvolution: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
