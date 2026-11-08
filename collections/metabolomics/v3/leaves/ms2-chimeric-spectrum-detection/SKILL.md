---
name: ms2-chimeric-spectrum-detection
description: Use when processing DDA LC-MS/MS data where multiple centroids fall inside
  an MS2 isolation window and contamination-ion fragments must be removed from co-isolated
  chimeric spectra before compound identification.
when_to_use_negative:
- Input is SWATH-DIA data — relinking precursors with fragment ions requires the separate
  PerformDIADeconvolution/reconstructChromPeakSpectra workflow instead
- MS2 spectra are already clean (no multiple centroids inside the isolation window
  above the acquisition threshold) — deconvolution would needlessly alter valid spectra
- Only an MS1 feature table is available with no associated MS2 spectra — chimeric
  detection requires per-feature MS2 scans
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformDDADeconvolution, which orchestrates chimeric-status evaluation,
    candidate component spectrum construction (Candidate Spectra I-IV), and the elastic-net
    regression deconvolution
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Reads centroided mzML raw data and accesses nearest MS scans for chimeric-status
    evaluation
- name: R
  role: Open-source environment in which the DDA deconvolution module runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:8c9920ae234d5bb9b9dfec08496bdb9caa01967e388e00f48cf524c95e09b9e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - mzR
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# ms2-chimeric-spectrum-detection

## Summary

Detects and deconvolves chimeric DDA MS2 spectra in MetaboAnalystR 4.0 by flagging co-isolated contamination ions within the mz_med-centered isolation window and using a self-tuned elastic-net regression to separate the main ion's clean spectrum.

## When to use

Use when processing DDA LC-MS/MS data where multiple centroids fall inside an MS2 isolation window and contamination-ion fragments must be removed from co-isolated chimeric spectra before compound identification.

## When NOT to use

- Input is SWATH-DIA data — relinking precursors with fragment ions requires the separate PerformDIADeconvolution/reconstructChromPeakSpectra workflow instead
- MS2 spectra are already clean (no multiple centroids inside the isolation window above the acquisition threshold) — deconvolution would needlessly alter valid spectra
- Only an MS1 feature table is available with no associated MS2 spectra — chimeric detection requires per-feature MS2 scans

## Inputs

- Centroided mzML raw data (read via mzR/ProteoWizard)
- Aligned MS1 feature table from the auto-optimized feature detection module
- MS2 reference spectral libraries (e.g., HMDB, MoNA, MassBank, GNPS, LipidBlast, MINEs)
- User parameters: precursor m/z ppm tolerance, rt_tol, isolation window, acquisition intensity threshold

## Outputs

- Normalized deconvoluted MS2 spectrum for each main ion of a chimeric feature
- Retained original 'Spectrum 0' when no fragment remains or the main ion's beta is 0
- Parseable deconvolution output file produced by PerformDDADeconvolution

## How to apply

Load centroided mzML data and the aligned MS1 feature table, then assign each MS2 spectrum to its target MS1 feature by precursor m/z (ppm tolerance) and retention time (rt_tol), merging replicate spectra in a weighted MZmine-style consensus. Mark a spectrum as chimeric when multiple centroids fall inside the mz_med-centered isolation window and any intensity exceeds the user-defined acquisition threshold, labeling the target feature's centroid as the 'main ion' and the rest as 'contamination ions'. Build candidate component spectra from four sources: DecoID-style prediction of orphan isotopologues from the MS1 scan, clean spectra of the one or two nearest MS2 scans, the highest dot-product/spectral-entropy 'Complete library' match, and NetID-rule-predicted spectra of unknown ions from biotransformation neighbors (with a 0-10 similarity penalty and removal of fragments whose element composition contradicts the unknown ion's formula). Deconvolve 'Spectrum 0' with a penalized elastic-net regression, permuting 110 alpha-lambda combinations (11 alpha values from 0 to 1 in steps of 0.1, 10 lambdas estimated from response-component correlations) and keeping the minimal-residue Solution 0, with alpha and lambda optimized per peak. Remove contamination-ion fragments using their beta coefficients, normalize the remaining fragments, and export the deconvoluted spectrum for the main ion.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformDDADeconvolution, which orchestrates chimeric-status evaluation, candidate component spectrum construction (Candidate Spectra I-IV), and the elastic-net regression deconvolution) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Reads centroided mzML raw data and accesses nearest MS scans for chimeric-status evaluation)
- **R** (Open-source environment in which the DDA deconvolution module runs)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- Every chimeric feature yields either a normalized deconvoluted spectrum or a retained original 'Spectrum 0' — no feature is left without an output
- The output file is parseable and was produced by the PerformDDADeconvolution routine
- The regression solution retained (Solution 0) has the minimal residue across all 110 permuted alpha-lambda combinations, with alpha and lambda optimized for every individual peak
- Contamination-ion fragments are absent from the exported spectrum, confirming removal via their beta coefficients
- If the main ion's beta coefficient is 0 or no fragment remains, the exported spectrum must equal the original 'Spectrum 0' (fallback path exercised)

## Limitations

- Over half of DDA MS2 spectra are typically chimeric, so the 110-combination elastic-net permutation per peak is computationally demanding
- Deconvolution silently falls back to the original chimeric 'Spectrum 0' when no fragment remains or the main ion's beta is 0, so downstream identification may use contaminated spectra
- Candidate Spectrum III quality depends on reference libraries compiled from various instruments and experimental conditions, resulting in potentially varying qualities

## Evidence

- [methods] The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution.: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [intro] an auto-optimized DDA data deconvolution workflow to deal with chimeric spectra: "an auto-optimized DDA data deconvolution workflow to deal with chimeric spectra"
- [intro] over half of them are chimeric and need to be deconvolved: "over half of them are chimeric and need to be deconvolved"
- [readme] a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA): "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
