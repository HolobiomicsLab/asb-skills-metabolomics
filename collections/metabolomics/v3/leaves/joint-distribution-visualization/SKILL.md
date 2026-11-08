---
name: joint-distribution-visualization
description: Use when comparing a spectral predictor's quality metric (e.g., cosine
  similarity) against a coverage-type covariate (annotated peak-intensity fraction)
  across two benchmark datasets to expose a bounding relationship and annotation-rate
  differences.
when_to_use_negative:
- The input is raw unextracted LC-MS/MS data rather than matched predicted-vs-observed
  spectra pairs — coverage and cosine cannot be computed before peak matching and
  consensus-spectrum extraction
- Only a single dataset is available with no second benchmark for marginal comparison
  — the skill's core output is the side-by-side joint distribution (e.g., CASMI 2016
  vs CASMI 2022)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: FIORA
  role: Graph neural network used via fiora-predict to generate the predicted spectra
    whose coverage-cosine relationship is visualized
  repo: https://github.com/BAMeScience/fiora
- name: fiora-predict
  role: CLI invoked with -i <casmi_input.csv> -o <predicted.mgf> --model default to
    produce mgf-format predicted spectra at all 3 collision energies for the stepped
    CASMI 2016 spectra
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:e1f753ee6b8ec5c5e14967184140b54f9ad2344f3a5d6d2bbc9540713601cfdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  - fiora-predict
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# joint-distribution-visualization

## Summary

Reproduces the FIORA paper's Fig. 6 analysis: joint (marginal-annotated) distributions of annotated peak-intensity coverage versus sqrt-cosine spectral similarity for FIORA predictions on CASMI 2016 and CASMI 2022, demonstrating that cosine similarity is bounded by the square root of intensity coverage.

## When to use

Use when comparing a spectral predictor's quality metric (e.g., cosine similarity) against a coverage-type covariate (annotated peak-intensity fraction) across two benchmark datasets to expose a bounding relationship and annotation-rate differences.

## When NOT to use

- The input is raw unextracted LC-MS/MS data rather than matched predicted-vs-observed spectra pairs — coverage and cosine cannot be computed before peak matching and consensus-spectrum extraction
- Only a single dataset is available with no second benchmark for marginal comparison — the skill's core output is the side-by-side joint distribution (e.g., CASMI 2016 vs CASMI 2022)

## Inputs

- Predicted spectra in mgf format from fiora-predict for CASMI 2016 (merged stepped-collision-energy spectra) and CASMI 2022
- Observed CASMI 2016 and CASMI 2022 challenge spectra (CASMI 22 extracted as consensus spectra with 10 ppm precursor tolerance, 5 s RT window)
- Input CSVs with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type'

## Outputs

- casmi_coverage_cosine_metrics.csv (per-spectrum coverage, cosine similarity, annotation pass/fail per dataset)
- fig6_coverage_cosine_joint_distribution.png (joint distributions of intensity coverage vs cosine similarity with marginal annotation rates and medians for both CASMI datasets)

## How to apply

First generate predicted MS/MS spectra per dataset with the fiora-predict CLI (for CASMI 2016, predict at all 3 stepped collision energies and merge into a single 'stepped' spectrum; for CASMI 2022, extract consensus spectra with 10 ppm precursor tolerance and a 5 s retention time window). Match predicted fragment peaks to observed peaks and compute per-spectrum annotated peak-intensity coverage, enforcing the paper's filters (at least two peak matches, minimum intensity coverage of 50%, maximum precursor intensity of 90%). Compute spectral_sqrt_cosine similarity per spectrum using FIORA's evaluation scoring, then numerically verify that each per-spectrum cosine value does not exceed the square root of its intensity coverage. Finally tabulate coverage, cosine, and annotation pass/fail into casmi_coverage_cosine_metrics.csv and plot joint distributions with marginal annotation rates and medians (expected ~50% vs ~25% annotation rates; ~80% vs ~32% median coverage for CASMI 2016 vs CASMI 2022).

## Related tools

- **FIORA** (Graph neural network used via fiora-predict to generate the predicted spectra whose coverage-cosine relationship is visualized) — https://github.com/BAMeScience/fiora
- **fiora-predict** (CLI invoked with -i <casmi_input.csv> -o <predicted.mgf> --model default to produce mgf-format predicted spectra at all 3 collision energies for the stepped CASMI 2016 spectra) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i casmi_input.csv -o predicted.mgf --model default
```

## Evaluation signals

- The inequality sqrt(coverage) >= cosine holds for every per-spectrum pair in both datasets
- Computed annotation rates fall near the reported ~50% (CASMI 2016) and ~25% (CASMI 2022)
- Computed median intensity coverages fall near the reported ~80% (CASMI 2016) and ~32% (CASMI 2022)
- Every included spectrum passes the filters: at least two peak matches, minimum intensity coverage of 50%, maximum precursor intensity of 90%
- Output CSV contains per-spectrum rows with coverage, cosine similarity, and annotation pass/fail columns for both datasets

## Limitations

- Low annotated peak-intensity coverage caps achievable cosine similarity (bounded by sqrt of coverage), so low-coverage datasets like CASMI 2022 (32% median coverage) score low regardless of model quality
- Not all collision energies for all CASMI 2022 challenge spectra were found during consensus extraction
- Only 62% of CASMI 2016 spectra reach cosine >0.7 even with >80% median coverage

## Evidence

- [other] Cosine similarity is bounded by the square root of intensity coverage; CASMI 16 (50% peak annotation rate, >80% median coverage, 62% of spectra with cosine >0.7) scores high, while CASMI 22 (25% annotation, 32% median coverage) scores low.: [evidence span withheld — anchor status: not_found]
- [discussion] We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%.: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single “stepped" spectrum.: "we predicted spectra with FIORA for all 3 collision energies and merged them into a single “stepped" spectrum"
- [discussion] Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds.: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [readme] By default, an open-source model is selected automatically, and predictions typically complete within a few seconds.: "By default, an open-source model is selected automatically, and predictions typically complete within a few seconds."
