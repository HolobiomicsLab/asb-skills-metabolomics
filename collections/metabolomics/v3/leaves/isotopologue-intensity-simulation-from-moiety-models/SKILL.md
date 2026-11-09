---
name: isotopologue-intensity-simulation-from-moiety-models
description: Use when you need Monte Carlo benchmarking of how measurement noise propagates
  through moiety-model-derived isotopologue profiles — e.g. comparing 13C-only vs
  13C/18O glucose multi-tracer error behavior at defined sigma levels.
when_to_use_negative:
- Input is real measured MS isotopologue data that needs deconvolution or moiety model
  fitting — use moiety_modeling optimization/model selection instead of forward simulation
- The goal is metabolic flux analysis from curated metabolic network models (e.g.
  EMU framework) rather than isotopologue-profile noise characterization
- You need model selection among candidate moiety models ranked by AICc; this skill
  assumes a fixed, already-specified model
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: moiety_modeling
  role: Provides moiety model representation from which Eq. 1 relative isotopologue
    intensities are computed as the noise-free baseline of the simulation
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Supplies the normally distributed random error added (thresholded) to the
    computed isotopologue intensities at sigma = 0.001, 0.01, 0.1
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the renormalized averaged simulation results (Python objects) to
    a JSON file
  repo: https://github.com/jsonpickle/jsonpickle
- name: matplotlib
  role: Used to plot the simulated averaged isotopologue results (Fig 6)
  repo: https://github.com/matplotlib/matplotlib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:c67afe4683c2b53e9bcbac933cd791774c359757d1ec31dd7188ccb8fcf39dc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - jsonpickle
  - matplotlib
  - moiety_modeling
  - scipy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isotopologue-intensity-simulation-from-moiety-models

## Summary

Generates simulated single- and multi-tracer isotopologue datasets by adding thresholded, normally distributed error to intensities computed from moiety models (Eq. 1), then averages isotopologue sums across replicates before renormalization to characterize error propagation (Table 8, Fig 6 of the moiety modeling paper).

## When to use

Use when you need Monte Carlo benchmarking of how measurement noise propagates through moiety-model-derived isotopologue profiles — e.g. comparing 13C-only vs 13C/18O glucose multi-tracer error behavior at defined sigma levels.

## When NOT to use

- Input is real measured MS isotopologue data that needs deconvolution or moiety model fitting — use moiety_modeling optimization/model selection instead of forward simulation
- The goal is metabolic flux analysis from curated metabolic network models (e.g. EMU framework) rather than isotopologue-profile noise characterization
- You need model selection among candidate moiety models ranked by AICc; this skill assumes a fixed, already-specified model

## Inputs

- Moiety model JSON definitions for the single-tracer (13C) model with rounded moiety state values from Table 6
- Moiety model JSON definitions for the multi-tracer (13C/18O glucose) model with rounded moiety state values from Table 7
- Noise level specification: sigma ∈ {0.001, 0.01, 0.1} and repetition = 100 simulated datasets per model/sigma combination

## Outputs

- Renormalized averaged isotopologue intensity profiles per model and sigma level (Table 8 values, Fig 6 curves)
- Pre-renormalization average sum of isotopologues per model and sigma, expected to be approximately 1
- JSON-serialized simulation results written via jsonpickle

## How to apply

Load the single-tracer (13C) and multi-tracer (13C/18O glucose) moiety model definitions from their JSON files, using the rounded moiety state values reported in Tables 6 and 7. Compute relative isotopologue intensities for each model with Eq. 1, which maps moiety abundances onto the expected isotopologue profile. For each model and each noise level (sigma = 0.001, 0.01, 0.1), generate 100 simulated datasets (repetition=100) by adding thresholded normal random error (drawn via scipy) so intensities never go negative. Sum isotopologues within each simulated dataset and average these sums across the 100 replicates BEFORE renormalization — averaging prior to renormalization is the key design choice that makes error propagation differences visible. Renormalize the averaged intensities and serialize to a JSON file with jsonpickle, as the paper does for saving Python objects in a JSON serializable form.

## Related tools

- **moiety_modeling** (Provides moiety model representation from which Eq. 1 relative isotopologue intensities are computed as the noise-free baseline of the simulation) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Supplies the normally distributed random error added (thresholded) to the computed isotopologue intensities at sigma = 0.001, 0.01, 0.1) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the renormalized averaged simulation results (Python objects) to a JSON file) — https://github.com/jsonpickle/jsonpickle
- **matplotlib** (Used to plot the simulated averaged isotopologue results (Fig 6)) — https://github.com/matplotlib/matplotlib

## Examples

```
import numpy as np, jsonpickle; err = np.random.normal(0, 0.01, intensities.shape); sim = np.maximum(intensities + err, 0); open('sim.json','w').write(jsonpickle.encode(sim))
```

## Evaluation signals

- Pre-renormalization average sum of isotopologues is approximately 1 for each model and sigma, with deviation scaling with the noise level
- At sigma = 0.001, propagated error is comparable between single-tracer and multi-tracer simulated datasets (Table 8 expectation)
- At larger error, single-tracer datasets at sigma = 0.1 show slightly LESS propagated error than multi-tracer datasets at sigma = 0.01
- No simulated isotopologue intensity is negative — thresholding of the normal error must be enforced for all 100 replicates
- Output JSON file round-trips correctly when deserialized with jsonpickle and contains both model profiles and sigma levels

## Limitations

- Thresholding the normal error makes the effective noise non-Gaussian, and the reported deviations are specific to averaging isotopologue sums across replicates BEFORE renormalization; a different order of operations changes the propagation behavior
- Simulations use rounded moiety state values from Tables 6 and 7, so absolute values may differ slightly from unrounded model parameters
- Findings cover only the tested sigma levels (0.001, 0.01, 0.1) and these two exemplar moiety models (13C UDP-GlcNAc-type and 13C/18O glucose)

## Evidence

- [methods] 100 replicates at three sigma levels: [evidence span withheld — anchor status: not_found]
- [methods] validation invariant of unit total intensity: [evidence span withheld — anchor status: not_found]
- [methods] JSON serialization of Python objects: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] jsonpickle role in the pipeline: [evidence span withheld — anchor status: external_doc]
