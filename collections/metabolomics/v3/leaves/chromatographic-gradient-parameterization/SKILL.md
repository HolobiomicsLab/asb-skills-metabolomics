---
name: chromatographic-gradient-parameterization
description: 'Use when you must define the candidate LC-MS gradient space before fitting
  the GP model: you have gradient-definition parameters (dimensionality, time points,
  mobile-phase settings) and need the constrained gridX/scaledX candidate set.'
when_to_use_negative:
- When raw LC-MS data (mzML/mzXML) must be processed and separation efficiency computed
  — that is the downstream evaluation stage (findTopSignals, sepEfficiency), not search-space
  generation.
- When experimental data are already available and the task is fitting the Gaussian-process
  regression model or computing the next gradient via computeNextGradient/computeSecondGradient.
- When a single predefined gradient needs to be scored; this skill only enumerates
  the constrained candidate space, it does not evaluate gradients.
tools:
- name: BAGO
  role: Provides the gpModel class and genSearchSpace(parameters) function that enumerate
    the constrained gradient search space (gridX, scaledX, gradientPct).
  repo: https://github.com/HuanLab/bago
- name: Python
  role: Runtime language for the bago package (Python 3.8 or later, installed via
    pip) used to run the search-space generation.
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:5f30d699474572e38913cd070252613bf54184e5b7b7ff4fb789a5317620fded
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/huaxuyu/bago
  repo_url: https://github.com/HuanLab/bago
  tools:
  - BAGO
  - Python
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chromatographic-gradient-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates the first-stage search space of plausible LC gradients for the BAGO framework by enumerating p-dimensional vectors of strong mobile-phase percentage over gradient time points, subject to monotonicity and total-eluting-power constraints. It produces the gridX/scaledX/gradientPct arrays that feed the downstream Gaussian-process Bayesian optimization.

## When to use

Use when you must define the candidate LC-MS gradient space before fitting the GP model: you have gradient-definition parameters (dimensionality, time points, mobile-phase settings) and need the constrained gridX/scaledX candidate set.

## When NOT to use

- When raw LC-MS data (mzML/mzXML) must be processed and separation efficiency computed — that is the downstream evaluation stage (findTopSignals, sepEfficiency), not search-space generation.
- When experimental data are already available and the task is fitting the Gaussian-process regression model or computing the next gradient via computeNextGradient/computeSecondGradient.
- When a single predefined gradient needs to be scored; this skill only enumerates the constrained candidate space, it does not evaluate gradients.

## Inputs

- BAGO parameter dictionary with gradient-definition parameters (gradient dimensionality, gradient time points, mobile-phase settings)
- Supplementary parameter settings used for expert review of the space cardinality

## Outputs

- gridX array of enumerated candidate gradient configurations
- scaledX array (scaled gradient coordinates)
- gradientPct attribute (strong mobile-phase percentage per gradient)
- Tabular search_space.csv file containing the search-space arrays

## How to apply

Read the gradient-definition parameters (gradient dimensionality, time points, mobile-phase settings) from the BAGO parameter dictionary and pass them to gpModel.genSearchSpace, which is implemented in the gpModel class to generate a search space. Enumerate candidate gradients as p-dimensional vectors of mobile-phase percentages over the gradient time points, producing the gridX and scaledX arrays plus the gradientPct attribute (strong mobile-phase percentage per gradient). During enumeration enforce the two defining constraints: strong mobile-phase percentage must be monotonically increasing along the gradient, and total eluting power must stay within a reasonable range. Check constraint compliance by computing getMobilePhasePct(gradient, timePoints) for each enumerated gradient. Write the resulting search-space arrays to a tabular file search_space.csv. Validate by asserting every emitted row is monotonic in strong mobile phase and within the eluting-power range; the space cardinality (over 100,000 gradients, 261,484 for one dataset) is accepted via expert review against the supplementary parameter settings.

## Related tools

- **BAGO** (Provides the gpModel class and genSearchSpace(parameters) function that enumerate the constrained gradient search space (gridX, scaledX, gradientPct).) — https://github.com/HuanLab/bago
- **Python** (Runtime language for the bago package (Python 3.8 or later, installed via pip) used to run the search-space generation.) — https://github.com/HuanLab/bago

## Examples

```
from bago import gpModel; model = gpModel(); gridX, scaledX = model.genSearchSpace(parameters)
```

## Evaluation signals

- Every emitted gradient row is monotonically increasing in strong mobile-phase percentage (assertion script over search_space.csv).
- Every emitted gradient stays within the stated reasonable range of total eluting power.
- getMobilePhasePct(gradient, timePoints) values are consistent with the gradientPct attribute for each enumerated gradient.
- Cardinality of the space matches the expert-reviewed expectation (over 100,000 gradients; 261,484 for one dataset) against supplementary parameter settings.
- gridX/scaledX shapes are consistent with the configured gradient dimensionality and number of time points, and search_space.csv is written without missing rows.

## Limitations

- The exact cardinality of the space (over 100,000 gradients; 261,484 for one dataset) is validated by expert_review against supplementary parameter settings rather than a fully automated check.
- Because the space contains over 100,000 plausible gradients, exhaustive experimental evaluation is infeasible; the space is only useful as input to the subsequent Bayesian-optimization stages.
- The bago package requires Python 3.8 or later; the Windows software and Python package are distributed separately (GitHub releases / PyPI).

## Evidence

- [methods] This function is implemented in the :class:`gpModel` class to generate a search space.: "This function is implemented in the :class:`gpModel` class to generate a search space."
- [abstract] From over 100,000 plausible gradients, BAGO locates the optimal LC gradient within ten sample analyses.: "From over 100,000 plausible gradients, BAGO locates the optimal LC gradient within ten sample analyses."
- [readme] _bago_ is a Python package for supporting the proposed Bayesian Optimization framework of LC gradient optimization.: [evidence span withheld — anchor status: external_doc]
- [methods] .. function:: getMobilePhasePct(gradient, timePoints): ".. function:: getMobilePhasePct(gradient, timePoints)"
