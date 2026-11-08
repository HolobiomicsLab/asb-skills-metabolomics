---
name: simulated-measurement-noise-modeling
description: Use when benchmarking an annotation or ranking pipeline where true reference
  values exist but no real measured masses are available; noise must mimic instrument
  accuracy.
when_to_use_negative:
- You already have real measured m/z values from acquired samples — simulate noise
  only when true query values are unavailable
- The ranking step relies on fragmentation spectra rather than mass-plus-RT; this
  simulation covers only mass and RT signals
- Your mass tolerance is asymmetric or non-Gaussian; the truncation scheme assumes
  a symmetric ppm window
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implements the Gaussian noise sampling, truncation, and mass-search simulation
    within the cmmrt package
  repo: https://github.com/constantino-garcia/cmmrt
- name: cmmrt
  role: Provides the meta-learned GP projection and z-score machinery that consumes
    the simulated mass queries
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:bdcd46d1bc66e029ca4f29f66aa52a0b67affa30619ba0b27e469c1974aa579a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# simulated-measurement-noise-modeling

## Summary

Inject controlled Gaussian noise into known reference values (here, exact masses) to simulate instrument measurement error, so that candidate-ranking evaluation pipelines (e.g. mass-filtered metabolite annotation with RT z-scores) can be benchmarked without real query spectra.

## When to use

Use when benchmarking an annotation or ranking pipeline where true reference values exist but no real measured masses are available; noise must mimic instrument accuracy.

## When NOT to use

- You already have real measured m/z values from acquired samples — simulate noise only when true query values are unavailable
- The ranking step relies on fragmentation spectra rather than mass-plus-RT; this simulation covers only mass and RT signals
- Your mass tolerance is asymmetric or non-Gaussian; the truncation scheme assumes a symmetric ppm window

## Inputs

- Exact masses of reference molecules (e.g. the 6,823 KEGG-numbered molecules of HMDB)
- Instrument mass tolerance in ppm (e.g. 10 ppm)
- Candidate-annotation database (DNN RT predictions for each reference molecule)

## Outputs

- Simulated query masses for each reference molecule (noisy exact masses within the ppm window)
- Mass-filtered candidate lists feeding the RT z-score ranking step
- Top-three annotation accuracy counts per chromatographic method and training-set size

## How to apply

For each molecule with a known exact mass, draw Gaussian noise with zero mean and sigma = instrument_tolerance/3 (e.g. 10/3 ppm for a 10 ppm mass search), add it to the true mass, and truncate the noisy value to lie within +/- the stated tolerance (10 ppm) so the correct molecule is always retained as a candidate. Run the downstream filter (accurate-mass search over the 6,823 KEGG-numbered HMDB molecules) and ranking (RT z-scores from GP-projected DNN RT predictions) on these simulated queries. Use the same noise model for every chromatographic method (FEM long, FEM orbitrap plasma, LIFE old, RIKEN) so comparisons across training-set sizes (10-50 molecules) reflect ranking skill rather than query-quality artifacts.

## Related tools

- **Python** (Implements the Gaussian noise sampling, truncation, and mass-search simulation within the cmmrt package) — https://github.com/constantino-garcia/cmmrt
- **cmmrt** (Provides the meta-learned GP projection and z-score machinery that consumes the simulated mass queries) — https://github.com/constantino-garcia/cmmrt

## Examples

```
python -c "import numpy as np; noisy = np.clip(true_mass + np.random.normal(0, 10/3e-6), true_mass*(1-10e-6), true_mass*(1+10e-6))"
```

## Evaluation signals

- Every simulated query must return the true molecule among its candidates (truncation guarantee)
- Global top-three accuracy should reach ~68% with 10 training molecules and ~70% with 50, exceeding the mass-only baselines (e.g. 59.57% for FEM long)
- Noisy masses must never deviate from the exact mass by more than the stated tolerance (10 ppm)
- Noise must be zero-mean across many queries; check the empirical mean deviation is close to 0 ppm
- Results per CM (FEM long, FEM orbitrap plasma, LIFE old, RIKEN) should be reproducible with a fixed random seed

## Limitations

- Truncation of the noise beyond +/-10 ppm forces the correct molecule to always be a candidate, so this simulation cannot measure the effect of true identity misses
- Sigma = tolerance/3 is a modeling assumption; other instruments or acquisition modes may not follow a Gaussian error profile
- The evaluation is restricted to mass queries returning more than three candidates; single-candidate or two-candidate queries are excluded

## Evidence

- [other] simulate an accurate mass search with 10 ppm tolerance by adding Gaussian noise: [evidence span withheld — anchor status: not_found]
- [abstract] 68% top-three accuracy from mass-filtered RT ranking: "in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
- [methods] the 6,823 molecules with KEGG number in the Human Metabolome DataBase (HMDB): [evidence span withheld — anchor status: not_found]
- [abstract] Python code available at the cmmrt repository: "Python code is available on GitHub at https://github.com/constantino-garcia/cmmrt"
