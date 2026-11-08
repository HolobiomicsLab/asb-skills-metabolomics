---
name: nmr-spectrum-probability-measure-representation
description: Use when quantifying components of 1H NMR mixture spectra that suffer
  from overlapping peaks, shifted resonances, low resolution, or distorted lineshapes
  where peak-based deconvolution methods fail.
when_to_use_negative:
- When the mixture spectrum contains no overlapping or shifted peaks and a peak-based
  method such as MANIQ or ACD/TP would suffice
- When the goal is absolute quantification without a reference spectrum in the library
  — the external-standard workflow requires algebraically adding a known-concentration
  substance spectrum to the mixture
- When spectra come from a different modality without adaptation — multidimensional
  generalizations (sliced Wasserstein, graph-flow representations) are listed as future
  work only
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Magnetstein
  role: 'Implements the optimal-transport quantification: represents spectra as probability
    distributions of signal intensity, solves the augmented Wasserstein linear program
    with κmixture/κcomponents penalties, and outputs component proportions'
  repo: https://gitlab.mimuw.edu.pl/ag181044/masserfront
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:72e775361ec7628fcad22d3388961b33a41238f469d57de7a1ce3635755d5ce1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - Magnetstein
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nmr-spectrum-probability-measure-representation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Represents 1H NMR mixture and library spectra as discrete probability measures of signal intensity over the chemical-shift axis so that component quantification can be cast as a Wasserstein-distance optimal-transport problem, as implemented in Magnetstein.

## When to use

Use when quantifying components of 1H NMR mixture spectra that suffer from overlapping peaks, shifted resonances, low resolution, or distorted lineshapes where peak-based deconvolution methods fail.

## When NOT to use

- When the mixture spectrum contains no overlapping or shifted peaks and a peak-based method such as MANIQ or ACD/TP would suffice
- When the goal is absolute quantification without a reference spectrum in the library — the external-standard workflow requires algebraically adding a known-concentration substance spectrum to the mixture
- When spectra come from a different modality without adaptation — multidimensional generalizations (sliced Wasserstein, graph-flow representations) are listed as future work only

## Inputs

- Mixture 1H NMR spectrum (signal intensity vs chemical shift, normalized as a discrete probability measure)
- Library of reference component spectra normalized as discrete probability measures
- Denoising penalty parameters κmixture and κcomponents

## Outputs

- Estimated relative concentrations of mixture components (estimated_proportions.csv)
- Average relative error (AvRE) against known ground-truth proportions

## How to apply

Normalize the mixture spectrum and each library component spectrum into discrete probability measures of signal intensity over the chemical-shift axis, as prescribed by the optimal-transport formulation of Magnetstein. Compute the Wasserstein distance between the mixture measure and each candidate component measure by solving the underlying transportation linear program. Formulate the augmented linear program that adds the auxiliary point ω (the 'vortex') receiving excess signal with denoising penalty κmixture, and penalty κcomponents for signals missing from the components; solve it jointly to estimate the proportions πi. Tune the penalties within the stated regime: κcomponents higher than κmixture, while guarding against small κmixture values that push the smaller component into the vortex ω and inflate error (as seen with a ~1:10 two-component mixture). Output estimated relative concentrations and evaluate trueness via average relative error (AvRE) against ground truth.

## Related tools

- **Magnetstein** (Implements the optimal-transport quantification: represents spectra as probability distributions of signal intensity, solves the augmented Wasserstein linear program with κmixture/κcomponents penalties, and outputs component proportions) — https://gitlab.mimuw.edu.pl/ag181044/masserfront

## Evaluation signals

- Estimated relative concentrations reproduce reported trueness, e.g. AvRE 0.0% for the two-component Experiment 1 and AvRE 3.6% under lineshape distortion (Experiment 4), as summarized in Table S1 with optimal κmixture and κcomponents
- Chosen κmixture is not so small that the smaller component is assigned to vortex ω, which produces large relative error
- κcomponents is set higher than κmixture, the regime that gave the best results
- Proportions πi from the augmented linear program are consistent (sum to 1) with unexplained mixture signal accounted for at the auxiliary point ω

## Limitations

- Small κmixture values cause Magnetstein to put the smaller component into vortex ω, causing large relative error in imbalanced mixtures (~1:10)
- Peak-shift robustness relies on optimal transport; spectra with peaks shifted by different solvents or wrong shimming need appropriate penalty tuning (Examples 8 and 9 in the Supporting Information)
- No version number and no repository URL/Zenodo DOI for the open-source tool are provided in the article; comparison-tool versions are also unspecified
- Higher-dimensional generalizations (sliced Wasserstein distance, graph-theory flow/network representations) are planned future work, not implemented

## Evidence
