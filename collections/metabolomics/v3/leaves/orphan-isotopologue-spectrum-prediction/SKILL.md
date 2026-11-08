---
name: orphan-isotopologue-spectrum-prediction
description: Use when deconvolving chimeric DDA MS2 spectra and the isolation window
  contains ions lacking their own measured MS2 spectrum — their component spectra
  must be predicted from MS1 scan peaks before elastic-net unmixing.
when_to_use_negative:
- The isolation-window ion already has a clean (non-chimeric) MS2 scan — use Candidate
  Spectrum II from the nearest MS2 scans instead of predicting from MS1
- Processing DIA/SWATH data, where deconvolution relinks precursors to fragment ions
  via PerformDIADeconvolution/reconstructChromPeakSpectra rather than MS1-based isotopologue
  prediction
- The MS2 spectrum is not chimeric (single centroid in the isolation window), so no
  component spectra are needed
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformDDADeconvolution, in which orphan-isotopologue prediction
    (DecoID-style, Candidate Spectrum I) is embedded as the first candidate-spectrum
    builder before elastic-net unmixing
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the deconvolution and candidate-spectrum
    prediction routines run
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Reads the centroided mzML raw data whose MS1 scans supply the isotopologue
    peaks used for orphan-spectrum prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:6e491780e887458451e0db56b046bc28a4d8253b29b88dc80106c4ee76160788
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - mzR
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# orphan-isotopologue-spectrum-prediction

## Summary

Predicts MS2 fragment spectra for orphan ions (MS1 features with no acquired MS2 spectrum) from the precursor's MS1 scan using DecoID-style isotopologue prediction, serving as Candidate Spectrum I in MetaboAnalystR 4.0's PerformDDADeconvolution chimeric-spectra deconvolution workflow.

## When to use

Use when deconvolving chimeric DDA MS2 spectra and the isolation window contains ions lacking their own measured MS2 spectrum — their component spectra must be predicted from MS1 scan peaks before elastic-net unmixing.

## When NOT to use

- The isolation-window ion already has a clean (non-chimeric) MS2 scan — use Candidate Spectrum II from the nearest MS2 scans instead of predicting from MS1
- Processing DIA/SWATH data, where deconvolution relinks precursors to fragment ions via PerformDIADeconvolution/reconstructChromPeakSpectra rather than MS1-based isotopologue prediction
- The MS2 spectrum is not chimeric (single centroid in the isolation window), so no component spectra are needed

## Inputs

- Centroided mzML raw data (MS1 and MS2 scans)
- Aligned MS1 feature table with precursor m/z, retention time, and mz_med
- Isolation window definition and user-defined chimeric acquisition threshold
- Chimeric 'Spectrum 0' with labeled 'main ion' and 'contamination ion' centroids

## Outputs

- Candidate Spectrum I: DecoID-style predicted MS2 fragment spectrum for each orphan ion within the isolation window
- Combined candidate-component matrix fed into the elastic-net deconvolution (alongside Candidates II-IV)

## How to apply

During PerformDDADeconvolution, after marking a spectrum chimeric (multiple centroids within the mz_med-centered isolation window with any intensity exceeding the acquisition threshold), assemble the candidate component spectra needed for unmixing 'Spectrum 0'. For ions inside the isolation window that have no MS2 scan of their own, predict their fragment spectra from the associated MS1 scan using the DecoID-style approach: treat the MS1 peaks (including isotopologues) as precursors and generate their expected fragment spectra to serve as Candidate Spectrum I. This predicted spectrum joins Candidate Spectrum II (from the nearest one or two clean MS2 scans), Candidate Spectrum III (the best-matching 'Complete library' spectrum by dot-product or spectral-entropy similarity), and Candidate Spectrum IV (NetID network-predicted spectra of unknown ions, with fragments removed whose element composition contradicts the unknown ion's formula and a 0-10 similarity penalty applied). The full candidate set is then decomposed against the chimeric spectrum via penalized elastic-net regression over 110 alpha-lambda combinations (11 alpha values 0-1 in steps of 0.1, 10 lambda values from response-component correlations), keeping the minimal-residue Solution 0.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformDDADeconvolution, in which orphan-isotopologue prediction (DecoID-style, Candidate Spectrum I) is embedded as the first candidate-spectrum builder before elastic-net unmixing) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the deconvolution and candidate-spectrum prediction routines run) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Reads the centroided mzML raw data whose MS1 scans supply the isotopologue peaks used for orphan-spectrum prediction)

## Examples

```
library(MetaboAnalystR); mSet <- InitDataObjects("pktable", "na", FALSE); mSet <- PerformDDADeconvolution(mSet)
```

## Evaluation signals

- Every chimeric feature must yield either a normalized deconvoluted spectrum or a retained original 'Spectrum 0' in the output of PerformDDADeconvolution
- The exported output file must be parseable and attributable to the PerformDDADeconvolution routine
- The elastic-net fit over the 110 alpha-lambda combinations must return a minimal-residue Solution 0 for every individual peak
- If no fragment remains after removing contamination-ion fragments via beta coefficients, or the main ion's beta is 0, the original 'Spectrum 0' must be retained and exported (fallback invariant)

## Limitations

- Orphan-ion spectra are predictions, not measurements — accuracy depends on the DecoID-style isotopologue model and the quality of the MS1 scan peaks
- Only applies within the DDA chimeric-spectra path; the SWATH-DIA pipeline uses a different deconvolution strategy
- Candidate Spectrum IV's network-predicted spectra require a 0-10 similarity penalty and removal of fragments whose element composition contradicts the unknown ion's formula, so prediction quality degrades when NetID neighbors are sparse
- Over half of DDA MS2 spectra can be chimeric, making runtime and prediction coverage a practical bottleneck for large files

## Evidence

- [other] Candidate Spectrum I by DecoID-style prediction of orphan isotopologues from the MS1 scan: [evidence span withheld — anchor status: not_found]
- [methods] The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution.: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [intro] over half of them are chimeric and need to be deconvolved: "over half of them are chimeric and need to be deconvolved"
- [intro] an auto-optimized DDA data deconvolution workflow to deal with chimeric spectra: "an auto-optimized DDA data deconvolution workflow to deal with chimeric spectra"
- [other] Load centroided mzML raw data and the aligned MS1 feature table; assign each MS2 spectrum to its target MS1 feature by precursor m/z: [evidence span withheld — anchor status: not_found]
