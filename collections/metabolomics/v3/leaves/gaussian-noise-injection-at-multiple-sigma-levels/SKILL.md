---
name: gaussian-noise-injection-at-multiple-sigma-levels
description: Use when benchmarking the noise robustness of computed isotopologue profiles
  (e.g., single-tracer 13C vs multi-tracer 13C/18O glucose moiety models) and you
  need simulated replicate datasets at several error magnitudes before renormalization.
when_to_use_negative:
- Input is real measured MS isotopologue data rather than a model-computed (Eq. 1)
  isotopologue profile — noise injection is a simulation benchmark, not a correction
  step for observed data
- The error model of interest is multiplicative, count-based (Poisson), or instrument-specific
  rather than additive and normally distributed
- The goal is moiety model optimization or AICc-based model ranking of experimental
  data (that is the moiety_modeling optimize/rank workflow, not the simulation benchmark)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scipy
  role: Source of the normal random error added (thresholded) to the computed isotopologue
    intensities during simulation
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the simulated/renormalized results to a JSON file for storage and
    later analysis
  repo: https://github.com/jsonpickle/jsonpickle
- name: moiety_modeling
  role: Provides the moiety model representation used to compute the expected isotopologue
    profiles (Eq. 1) that noise is injected into
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:73321939615a9c9bd7923ec79c2907c395b9ba99bb899f943f7368b41e1f58fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scipy/scipy
  tools:
  - jsonpickle
  - moiety_modeling
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# gaussian-noise-injection-at-multiple-sigma-levels

## Summary

Simulates measurement noise on model-predicted isotopologue intensities by adding thresholded, normally distributed random error at escalating sigma levels (0.001, 0.01, 0.1) across 100 replicate datasets, to benchmark how error propagates in single-tracer versus multi-tracer moiety models.

## When to use

Use when benchmarking the noise robustness of computed isotopologue profiles (e.g., single-tracer 13C vs multi-tracer 13C/18O glucose moiety models) and you need simulated replicate datasets at several error magnitudes before renormalization.

## When NOT to use

- Input is real measured MS isotopologue data rather than a model-computed (Eq. 1) isotopologue profile — noise injection is a simulation benchmark, not a correction step for observed data
- The error model of interest is multiplicative, count-based (Poisson), or instrument-specific rather than additive and normally distributed
- The goal is moiety model optimization or AICc-based model ranking of experimental data (that is the moiety_modeling optimize/rank workflow, not the simulation benchmark)

## Inputs

- Moiety model definitions for the single-tracer (13C) and multi-tracer (13C/18O glucose) models as JSON files, with rounded moiety state values from Tables 6 and 7
- Relative isotopologue intensities computed from Eq. 1 for each model
- Sigma levels to test: 0.001, 0.01, 0.1
- Repetition count: 100 simulated datasets per model per sigma

## Outputs

- 100 simulated isotopologue datasets per model per sigma level with thresholded normal error added
- Pre-renormalization average sum of isotopologues per model and sigma (Table 8 / Fig 6)
- Renormalized averaged isotopologue intensities serialized to a JSON file via jsonpickle

## How to apply

Start from the moiety model definitions with rounded moiety state values (Tables 6 and 7) and compute relative isotopologue intensities via Eq. 1, which maps moiety abundances onto the expected isotopologue profile. For each model, generate 100 simulated datasets (repetition=100) by adding normal random error to the computed isotopologue intensities at sigma = 0.001, 0.01, and 0.1, thresholding so no intensity goes negative. Within each simulated dataset, sum the isotopologues, then average these sums across the 100 replicates for each model and sigma level prior to renormalization. Renormalize the averaged isotopologue intensities and serialize the results to a JSON file with jsonpickle. The escalating sigma ladder is the decision point: comparable propagation at sigma = 0.001 demonstrates equivalence at low noise, while divergence at higher sigma (single-tracer at sigma = 0.1 showing slightly less propagated error than multi-tracer at sigma = 0.01) characterizes multi-tracer sensitivity.

## Related tools

- **scipy** (Source of the normal random error added (thresholded) to the computed isotopologue intensities during simulation) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the simulated/renormalized results to a JSON file for storage and later analysis) — https://github.com/jsonpickle/jsonpickle
- **moiety_modeling** (Provides the moiety model representation used to compute the expected isotopologue profiles (Eq. 1) that noise is injected into) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling

## Examples

```
from scipy.stats import norm; err = norm.rvs(scale=0.1, size=len(intensities)); sim = intensities + err; sim[sim < 0] = 0
```

## Evaluation signals

- Pre-renormalization average sum of isotopologues is approximately 1 (within the deviation expected from the noise level) for every model and sigma, confirming the simulation preserves total intensity
- Each model x sigma combination (2 models x 3 sigmas) contains exactly 100 simulated replicate datasets (repetition=100)
- No simulated isotopologue intensity is negative — thresholding of the added normal error is verified
- At sigma = 0.001, propagated error is comparable between single-tracer and multi-tracer datasets, matching the reported finding
- Output JSON loads via jsonpickle with the expected structure of averaged, renormalized isotopologue intensities

## Limitations

- Error propagation deviates between tracer types at larger error levels: single-tracer datasets at sigma = 0.1 show slightly less propagated error than multi-tracer datasets at sigma = 0.01, so low-noise conclusions do not extrapolate to high noise
- Thresholding the normal error truncates the distribution, introducing bias that grows with sigma
- Only three sigma levels (0.001, 0.01, 0.1) and 100 replicates were tested, limiting resolution of the sigma-dependence curve

## Evidence

- [methods] pickle for JSON serialization of results: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] jsonpickle purpose: [evidence span withheld — anchor status: external_doc]
- [intro] multi-tracer complexity motivating the single- vs multi-tracer simulation comparison: "the simultaneous use of multiple stable isotopes in SIRM experiments can provide much more data than a single tracer."
- [abstract] scipy used for the simulation machinery: "Additional optimization methods from the Python scipy library are utilized as well"
