---
name: peak-splitting-annotation
description: Use when SAND output peak tables contain multiple fitted peaks per compound
  (split peaks, e.g., around 7.3 and 7.95 ppm in the simulated benchmark) and peaks
  must be mapped back to their compound of origin for accurate quantification.
when_to_use_negative:
- Spectra are not high-resolution metabolomics 1D NMR data — SAND's current version
  is tuned only for high-resolution metabolomics spectra and does not yet accommodate
  other cases
- Peaks are already provided as a fully annotated compound-level feature table with
  no splitting, so correlation-network clustering adds no value
- Input is multidimensional or nonuniformly sampled data — SAND is not yet adapted
  to multidimensional applications
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: SAND
  role: Produces the fitted peak tables via time-domain MCMC hybrid modeling that
    are then annotated and clustered by compound
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud environment where SAND runs to generate peak tables for annotation;
    updates must pass NMRbox tests
  repo: https://nmrbox.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:fa8224c76c88077ed22f42db4c2b0251cbc5dcc1b561beb58abd4d32403a5664
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - C/C++
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# peak-splitting-annotation

## Summary

Annotate SAND-derived peak tables by grouping fitted peaks into compound-of-origin clusters using correlation networks, recovering peaks that SAND's time-domain fit splits across positions (e.g., near 7.3 and 7.95 ppm) so each compound's concentration is estimated from its full peak set.

## When to use

Use when SAND output peak tables contain multiple fitted peaks per compound (split peaks, e.g., around 7.3 and 7.95 ppm in the simulated benchmark) and peaks must be mapped back to their compound of origin for accurate quantification.

## When NOT to use

- Spectra are not high-resolution metabolomics 1D NMR data — SAND's current version is tuned only for high-resolution metabolomics spectra and does not yet accommodate other cases
- Peaks are already provided as a fully annotated compound-level feature table with no splitting, so correlation-network clustering adds no value
- Input is multidimensional or nonuniformly sampled data — SAND is not yet adapted to multidimensional applications

## Inputs

- SAND peak tables (fitted peak positions, amplitudes, widths) from 1D 1H NMR spectra
- Simulated or experimental NMR spectra with known compound composition (e.g., ~100 overlapping peaks over a 1.50 ppm region, lambda 1-15)
- Ground-truth compound-to-peak mapping for validation

## Outputs

- Annotated peak table mapping each fitted SAND peak to a compound-of-origin cluster
- Cluster-level compound concentration estimates
- Correlation and slope metrics (target ~0.9 Pearson correlation, k ~0.9) against ground truth

## How to apply

First run the SAND time-domain modeling workflow (hybrid Markov chain Monte Carlo optimization with random training/validation subsampling of the time-domain data) on the 1D 1H NMR spectrum to obtain fitted peak tables. Because SAND's exponential model can split a compound's signal into multiple fitted peaks, apply automated annotation with correlation networks to cluster peaks belonging to the same compound; on the simulated benchmark (~100 peaks over a 1.50 ppm region, lambda 1-15) this recovered on average 74% of each compound's peaks in single clusters. Map each fitted peak to its ground-truth compound of origin, record splitting behavior around known regions such as 7.3 and 7.95 ppm, and aggregate cluster-level concentrations. Validate by computing the Pearson correlation and linear-regression slope (k) of SAND-estimated versus ground-truth concentrations; a correlation near 0.9 (reported 0.88) on highly overlapped simulated data indicates correct annotation and quantification.

## Related tools

- **SAND** (Produces the fitted peak tables via time-domain MCMC hybrid modeling that are then annotated and clustered by compound) — https://github.com/edisonomics/SAND
- **NMRbox** (Cloud environment where SAND runs to generate peak tables for annotation; updates must pass NMRbox tests) — https://nmrbox.org

## Evaluation signals

- On the simulated benchmark (~100 peaks, 1.50 ppm region), Pearson correlation of SAND-estimated vs ground-truth concentrations reaches approximately 0.9 (reported 0.88) with linear-regression slope k around 0.9
- On average 74% of peaks for each compound are recovered in single clusters via automated correlation-network annotation
- Peak-splitting behavior around 7.3 and 7.95 ppm is detected and recorded in the annotation output
- Each fitted peak in the output table is mapped to exactly one compound-of-origin cluster
- Annotation results are consistent between simulated and experimental datasets (urine, worm, two-compound mixture) using the same workflow

## Limitations

- SAND can split a compound's signal into multiple peaks; only ~74% of peaks per compound are recovered in single clusters, leaving some peaks ungrouped
- Current SAND implementation is tuned for high-resolution metabolomics spectra only and is a slow, non-parallelizable MATLAB version pending a C/C++ rewrite
- Autophasing or manual phase adjustment is preferred before fitting but not always needed, so unphased input may degrade peak-fitting reliability
- Reproducible quantification via the NMRbox/NAN workflow is still in progress and not fully realized

## Evidence

- [abstract] correlation-network annotation: [evidence span withheld — anchor status: not_found]
