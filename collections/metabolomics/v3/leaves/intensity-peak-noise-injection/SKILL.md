---
name: intensity-peak-noise-injection
description: Use when a ViMMS ChemicalMixtureCreator-generated chemical dataset needs
  realistic intensity fluctuation (and optionally random RT jitter) applied via pluggable
  noise classes before running simulated acquisition.
when_to_use_negative:
- The chemicals are still being generated from HMDB sampling or mzML ROI extraction
  and the base (unperturbed) dataset does not exist yet
- The goal is fragmenting/simulating acquisition (Top-N DDA, SmartROI, WeightedDEW)
  rather than perturbing chemical properties
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: 'Provides the pluggable noise classes (vimms.Noise: GaussianPeakNoise, GaussianPeakNoiseLevelSpecific,
    RetentionTimeDeviation) applied to simulated chemical mixtures'
  repo: https://github.com/glasgowcompbio/vimms
- name: numpy
  role: Core dependency used by ViMMS for numerical sampling underlying the noise
    models
- name: pandas
  role: Core dependency of the ViMMS framework supporting data handling during chemical
    perturbation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:f39970e3a0f5af38024886de7a6ae6542993c72f46741ff1fddcb1d301b98029
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Intensity Peak Noise Injection

## Summary

Injects pluggable peak-intensity noise (e.g. GaussianPeakNoise / GaussianPeakNoiseLevelSpecific from vimms.Noise) into ViMMS 2.0 chemical datasets so simulated LC-MS mixtures carry realistic signal-intensity fluctuation before controller-based MS/MS simulation.

## When to use

Use when a ViMMS ChemicalMixtureCreator-generated chemical dataset needs realistic intensity fluctuation (and optionally random RT jitter) applied via pluggable noise classes before running simulated acquisition.

## When NOT to use

- The chemicals are still being generated from HMDB sampling or mzML ROI extraction and the base (unperturbed) dataset does not exist yet
- The goal is fragmenting/simulating acquisition (Top-N DDA, SmartROI, WeightedDEW) rather than perturbing chemical properties

## Inputs

- Pickled ViMMS chemical dataset (master list of Chemical objects from ChemicalMixtureCreator, e.g. task_001.expected_outputs[0])
- Noise-model configuration (e.g. Gaussian level parameters for GaussianPeakNoiseLevelSpecific)

## Outputs

- Perturbed chemical dataset (e.g. perturbed_chemicals.pkl) where every chemical carries applied offset/drift/noise annotations
- Chemicals ready for simulation via controllers in an Environment

## How to apply

Load the master list of Chemical objects produced by ChemicalMixtureCreator (e.g. the pickled dataset from the chemical-generation step) and import the noise classes from vimms.Noise — GaussianPeakNoise or GaussianPeakNoiseLevelSpecific for intensity perturbation, optionally alongside UniformSpikeNoise. Apply the noise to the chemicals/spectra, typically combined with RetentionTimeDeviation for random RT jitter and column classes (CleanColumn wrapped in LinearColumn) for systematic RT offset/drift that emulates column ageing. Rebuild the chemical objects carrying the perturbed chromatograms and persist them (e.g. perturbed_chemicals.pkl) for use in downstream Environment/controller simulations. ViMMS 2.0 deliberately exposes this behaviour as pluggable, swappable noise modules so different noise models can be substituted without changing the simulation pipeline.

## Related tools

- **ViMMS 2.0** (Provides the pluggable noise classes (vimms.Noise: GaussianPeakNoise, GaussianPeakNoiseLevelSpecific, RetentionTimeDeviation) applied to simulated chemical mixtures) — https://github.com/glasgowcompbio/vimms
- **numpy** (Core dependency used by ViMMS for numerical sampling underlying the noise models)
- **pandas** (Core dependency of the ViMMS framework supporting data handling during chemical perturbation)

## Examples

```
from vimms.Noise import GaussianPeakNoiseLevelSpecific, RetentionTimeDeviation; perturbed = [GaussianPeakNoiseLevelSpecific.apply(c) for c in chemicals]
```

## Evaluation signals

- perturbed_chemicals.pkl exists and can be reloaded (pickle load succeeds)
- Every chemical in the reloaded dataset carries the applied offset/drift/noise annotations (structural completeness check)
- Perturbed chemicals still load into ChemicalMixtureCreator-style simulation and can be consumed by controllers in an Environment
- Noise classes are imported from vimms.Noise and are substitutable (pluggability invariant, per the modularity finding)

## Limitations

- No quantitative benchmark for the noise modules is reported in the paper; acceptance is structural completeness only
- Validation of ViMMS 2.0's utility is deferred to a separate publication (Davies et al., 2021), not presented in this paper
- High chemical counts raise processing cost: lowering min_intensity thresholds (e.g. from 1.75E5 to 1E4 or 0) includes more objects, making simulation much slower with higher memory usage

## Evidence

- [full_text] pluggable noise classes from vimms.Noise: [evidence span withheld — anchor status: not_found]
- [methods] ChemicalMixtureCreator produces chemical datasets: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [abstract] chemical objects from synthetic or mzML sources: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
- [abstract] swappable modular implementations: "The improved modularity in ViMMS 2.0 allows many aspects
of the framework to be swapped out with alternative implementations"
- [readme] simulate or replay LC-MS/MS: [evidence span withheld — anchor status: external_doc]
