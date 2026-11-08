---
name: transformation-network-spectrum-prediction
description: Use when a chimeric DDA MS2 spectrum contains contamination ions that
  have no clean reference or neighbor MS2 scan, so their component spectra must be
  predicted from transformation-network neighbors before elastic-net deconvolution.
when_to_use_negative:
- Input is SWATH-DIA data — use PerformDIADeconvolution / reconstructChromPeakSpectra
  instead of the DDA chimeric-spectrum path
- A clean spectrum is already observable for the contamination ion from one or two
  nearest MS2 scans, or a library spectrum with high dot-product/spectral-entropy
  similarity exists (Candidate II/III) — prediction is unnecessary and adds the 0-10
  penalty uncertainty
- The input is already a deconvoluted, normalized MS2 spectrum heading to PerformDBSearchingBatch
  for identification
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformDDADeconvolution and the Candidate Spectrum IV construction
    (NetID-rule transformation network, formula-consistency filtering, 0-10 similarity
    penalty) within the elastic-net deconvolution
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the DDA deconvolution module and its regression/permutation
    logic are executed
- name: mzR
  role: Reads centroided mzML raw MS2 spectra that feed the chimeric-spectrum grouping
    and candidate construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:f2265f29c0c7ff5e53dcb146a7c09bc8a8eaaf6b3d7b2e522ddb63274f0de99d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - mzR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# transformation-network-spectrum-prediction

## Summary

Predicts candidate MS2 spectra for unknown contaminant ions inside chimeric DDA spectra using biotic/abiotic transformation neighbors in a NetID-rule, non-propagatable network (Candidate Spectrum IV in MetaboAnalystR 4.0's PerformDDADeconvolution), enabling deconvolution when no directly observed clean spectrum exists.

## When to use

Use when a chimeric DDA MS2 spectrum contains contamination ions that have no clean reference or neighbor MS2 scan, so their component spectra must be predicted from transformation-network neighbors before elastic-net deconvolution.

## When NOT to use

- Input is SWATH-DIA data — use PerformDIADeconvolution / reconstructChromPeakSpectra instead of the DDA chimeric-spectrum path
- A clean spectrum is already observable for the contamination ion from one or two nearest MS2 scans, or a library spectrum with high dot-product/spectral-entropy similarity exists (Candidate II/III) — prediction is unnecessary and adds the 0-10 penalty uncertainty
- The input is already a deconvoluted, normalized MS2 spectrum heading to PerformDBSearchingBatch for identification

## Inputs

- Centroided mzML raw data (via ProteoWizard/mzR)
- Aligned MS1 feature table with precursor m/z and retention time
- Chimeric 'Spectrum 0' assignments (main ion vs contamination ions from the isolation window)
- Transformation knowledgebase: NetID rules and biotic/abiotic transformation pairs (non-propagatable network)
- Element-composition/formula annotations for unknown ions and their fragments

## Outputs

- Predicted candidate component spectrum for the unknown contamination ion (Candidate Spectrum IV) with 0-10 similarity penalty
- Beta coefficients per fragment used in the elastic-net deconvolution
- Deconvoluted, normalized MS2 spectrum for the main ion (or the retained original 'Spectrum 0') exported as a parseable file

## How to apply

Within MetaboAnalystR 4.0's DDA deconvolution workflow (PerformDDADeconvolution), first confirm chimeric status: multiple centroids inside the mz_med-centered isolation window with intensity above the user-defined acquisition threshold, one labeled 'main ion', the rest 'contamination ions'. For a contamination ion lacking Candidates I-III (DecoID-predicted isotopologues, nearest-MS2-scan clean spectra, or the best dot-product/spectral-entropy library match), apply NetID rules on a non-propagatable network to find its abiotic/bio-transformation neighbors and predict its spectrum (Candidate Spectrum IV). Remove predicted fragments whose element composition contradicts the unknown ion's formula, and apply a 0-10 similarity penalty to reflect prediction uncertainty. The predicted spectrum then enters the penalized elastic-net regression (110 alpha-lambda combinations, minimal-residue Solution 0), and the contamination ion's beta coefficients are used to strip its fragments from the chimeric 'Spectrum 0' before normalization and export.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformDDADeconvolution and the Candidate Spectrum IV construction (NetID-rule transformation network, formula-consistency filtering, 0-10 similarity penalty) within the elastic-net deconvolution) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the DDA deconvolution module and its regression/permutation logic are executed)
- **mzR** (Reads centroided mzML raw MS2 spectra that feed the chimeric-spectrum grouping and candidate construction)

## Examples

```
mSet <- PerformDDADeconvolution(mSet)
```

## Evaluation signals

- Every chimeric feature yields either a normalized deconvoluted spectrum for the main ion or a retained original 'Spectrum 0' — no feature is dropped silently
- The exported deconvolution output file is parseable and attributable to the PerformDDADeconvolution routine
- Predicted Candidate Spectrum IV fragments all have element compositions consistent with the unknown ion's formula (contradicting fragments removed)
- Similarity penalties assigned to network-predicted spectra stay within the defined 0-10 range
- If the main ion's beta is 0 or no fragment remains after stripping contamination-ion betas, the original Spectrum 0 is correctly retained rather than an empty spectrum exported

## Limitations

- If no fragment remains after removing contamination-ion fragments, or the main ion's beta coefficient is 0, the pipeline falls back to retaining the original chimeric 'Spectrum 0', which still contains contamination
- Network prediction applies only when NetID-rule transformation neighbors exist; ions without abiotic/bio-transformation neighbors cannot get Candidate Spectrum IV
- The 0-10 similarity penalty on predicted spectra is a heuristic uncertainty measure, not a validated confidence score; built-in reference spectra libraries are compiled from varying instruments and experimental conditions with potentially varying qualities

## Evidence

- [supplementary] Candidate Spectrum IV by predicting spectra of unknown ions from abiotic/bio-transformation neighbors (NetID rules, non-propagatable network), removing fragments whose element composition contradicts the unknown ion's formula and applying a 0-10 similarity penalty.: [evidence span withheld — anchor status: not_found]
- [methods] The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution.: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [intro] over half of them are chimeric and need to be deconvolved: "over half of them are chimeric and need to be deconvolved"
- [readme] a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA): "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
