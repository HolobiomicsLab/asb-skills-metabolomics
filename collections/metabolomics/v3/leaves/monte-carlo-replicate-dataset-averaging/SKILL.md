---
name: monte-carlo-replicate-dataset-averaging
description: Use when you have computed (noise-free) relative isotopologue intensities
  from a moiety model (single 13C tracer or 13C/18O multi-tracer) and need to quantify
  how additive measurement error at multiple sigma levels propagates into summed isotopologue
  totals prior to renormalization.
when_to_use_negative:
- When the goal is fitting or optimizing moiety model parameters against real measured
  isotopologues — use moiety_modeling optimize with SAGA-optimize/scipy instead of
  Monte Carlo simulation
- When the input is raw MS isotopologue data rather than a computed noise-free isotopologue
  profile from a moiety model — there are no computed intensities to perturb
- When real experimental replicate datasets exist, since measured replicates supersede
  simulated noise-based averaging
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: moiety_modeling
  role: Provides the moiety model definitions and Eq. 1 isotopologue intensity computation
    that form the noise-free baseline being perturbed
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Source of the normally distributed random additive error added to isotopologue
    intensities at sigma = 0.001, 0.01, 0.1
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the averaged and renormalized simulation results (Python objects)
    to a JSON file
  repo: https://github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:75b5c9c84068ae51e9c4a91ebccccb389ac431e10681f73f4d0a55e509108165
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - jsonpickle
  - moiety_modeling
  - scipy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# monte-carlo-replicate-dataset-averaging

## Summary

Monte Carlo simulation of single- and multi-tracer isotopologue datasets (100 replicates per model per noise level) by adding thresholded normally distributed additive error to computed isotopologue intensities, averaging the sums of isotopologues across replicates before renormalization, to characterize how measurement error propagates through moiety-model-derived isotopologue profiles.

## When to use

Use when you have computed (noise-free) relative isotopologue intensities from a moiety model (single 13C tracer or 13C/18O multi-tracer) and need to quantify how additive measurement error at multiple sigma levels propagates into summed isotopologue totals prior to renormalization.

## When NOT to use

- When the goal is fitting or optimizing moiety model parameters against real measured isotopologues — use moiety_modeling optimize with SAGA-optimize/scipy instead of Monte Carlo simulation
- When the input is raw MS isotopologue data rather than a computed noise-free isotopologue profile from a moiety model — there are no computed intensities to perturb
- When real experimental replicate datasets exist, since measured replicates supersede simulated noise-based averaging

## Inputs

- Single-tracer (13C) moiety model JSON with rounded moiety state values (Table 6)
- Multi-tracer (13C/18O glucose) moiety model JSON with rounded moiety state values (Table 7)
- Computed relative isotopologue intensity profile from Eq. 1
- Simulation parameters: repetition=100, sigma ∈ {0.001, 0.01, 0.1}

## Outputs

- Per-model, per-sigma averaged (over 100 replicates) pre-renormalization sums of isotopologues
- Renormalized averaged isotopologue intensity profiles
- JSON-serialized simulation results via jsonpickle
- Error-propagation comparison table (Table 8) and figure (Fig 6)

## How to apply

Load the single-tracer (13C) and multi-tracer (13C/18O glucose) moiety model definitions from the model JSON files, using the rounded moiety state values from Tables 6 and 7, then compute relative isotopologue intensities via Eq. 1. For each model, draw thresholded normal random additive error with scipy at sigma = 0.001, 0.01, and 0.1 and add it to the computed intensities, repeating 100 times (repetition=100) per model and sigma. Within each simulated dataset, sum the isotopologues, then average these sums across the 100 replicates for each model and sigma BEFORE renormalizing the averaged intensities — averaging prior to renormalization is the key order-of-operations choice, since noise makes raw sums deviate from 1. Renormalize the averaged intensities and serialize the results with jsonpickle. Interpret propagation by comparing pre-renormalization deviations: at sigma = 0.001 single- and multi-tracer propagation is comparable, while at larger sigma single-tracer datasets (sigma = 0.1) show slightly less propagated error than multi-tracer datasets (sigma = 0.01).

## Related tools

- **moiety_modeling** (Provides the moiety model definitions and Eq. 1 isotopologue intensity computation that form the noise-free baseline being perturbed) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Source of the normally distributed random additive error added to isotopologue intensities at sigma = 0.001, 0.01, 0.1) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the averaged and renormalized simulation results (Python objects) to a JSON file) — https://github.com/jsonpickle/jsonpickle

## Examples

```
from scipy.stats import norm; sim = [max(0, obs + r) for obs, r in zip(intensities, norm.rvs(loc=0, scale=0.01, size=len(intensities)))]
```

## Evaluation signals

- Verify the pre-renormalization average sum of isotopologues is approximately 1 (within the deviation expected from the noise level) for each model and sigma, confirming the simulation preserves total intensity
- Confirm 100 replicates were generated per model per sigma (repetition=100) before averaging
- Reproduce the paper's relationship: comparable error propagation between single- and multi-tracer datasets at sigma = 0.001, with single-tracer at sigma = 0.1 showing slightly less propagated error than multi-tracer at sigma = 0.01
- Check that the jsonpickle-serialized output deserializes back to the renormalized averaged intensity profiles matching Tables 8 / Fig 6

## Limitations

- Error is thresholded and normally distributed — this additive-error model may not capture multiplicative or intensity-dependent measurement noise
- Only sigma = 0.001, 0.01, and 0.1 were tested; propagation behavior at untested noise levels or error distributions is uncharacterized
- The paper notes future work is needed to 'explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection', so simulation-based conclusions on multi-tracer behavior remain preliminary

## Evidence

- [abstract] scipy for simulation: "Additional optimization methods from the Python scipy library are utilized as well"
- [methods] pickle for JSON serialization: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] jsonpickle README description: [evidence span withheld — anchor status: not_found]
- [discussion] future validation of multi-tracer analysis: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
