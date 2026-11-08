---
name: coverage-cosine-bound-verification
description: Use when benchmarking FIORA-predicted MS/MS spectra against observed
  CASMI 2016/CASMI 2022 spectra and you need to quantify how annotated peak-intensity
  coverage bounds cosine similarity per spectrum.
when_to_use_negative:
- The goal is benchmarking FIORA against ICEBERG/CFM-ID prediction quality rather
  than analyzing the coverage–cosine relationship — use the CASMI scoring comparison
  workflow instead
- Spectra were measured with multi-stage fragmentation (MSn), which FIORA does not
  support (single-step fragmentation is its biggest limitation)
- You only need retention-time or CCS predictions — this skill is specific to MS/MS
  spectral cosine and intensity-coverage analysis
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Graph neural network whose fiora-predict CLI generates the predicted mgf spectra
    whose cosine similarity and intensity coverage are computed and bound-checked
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:dfb7ad85482076078e0e8598cf9d4c2ea4d1f66c13c9941af91a906e842db186
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  - fiora-predict
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# coverage-cosine-bound-verification

## Summary

Verifies that per-spectrum spectral_sqrt_cosine similarity of FIORA predictions never exceeds the square root of the annotated peak-intensity coverage, and tabulates coverage–cosine joint distributions on the CASMI 2016 and CASMI 2022 challenge sets (reproducing Fig. 6).

## When to use

Use when benchmarking FIORA-predicted MS/MS spectra against observed CASMI 2016/CASMI 2022 spectra and you need to quantify how annotated peak-intensity coverage bounds cosine similarity per spectrum.

## When NOT to use

- The goal is benchmarking FIORA against ICEBERG/CFM-ID prediction quality rather than analyzing the coverage–cosine relationship — use the CASMI scoring comparison workflow instead
- Spectra were measured with multi-stage fragmentation (MSn), which FIORA does not support (single-step fragmentation is its biggest limitation)
- You only need retention-time or CCS predictions — this skill is specific to MS/MS spectral cosine and intensity-coverage analysis

## Inputs

- Input CSV with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' for CASMI 2016 (stepped CEs) and CASMI 2022
- Observed CASMI 2022 consensus spectra extracted with 10 ppm precursor tolerance and 5 s RT window
- FIORA default model checkpoint packaged under fiora/resources/models

## Outputs

- casmi_coverage_cosine_metrics.csv with per-spectrum intensity coverage, spectral_sqrt_cosine value, and annotation pass/fail per dataset
- fig6_coverage_cosine_joint_distribution.png showing joint coverage-vs-cosine distributions with marginal annotation rates and medians
- Predicted spectra in mgf format (merged 'stepped' spectra for CASMI 2016)

## How to apply

Run fiora-predict (-i input CSV with 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' columns, --model default) to produce predicted spectra in mgf; for CASMI 2016 predict at all 3 stepped collision energies and merge into a single 'stepped' spectrum, for CASMI 2022 use consensus spectra extracted with 10 ppm precursor tolerance and a 5 s RT window. Match predicted fragments to observed peaks and compute per-spectrum annotated peak-intensity coverage under the paper's hard filters: at least two peak matches, minimum intensity coverage of 50%, and maximum precursor intensity of 90% (soft filters: 50% of total peaks or ≥5 peaks matched, or exceptionally high intensity coverage of 0.8). Compute spectral_sqrt_cosine for each spectrum and numerically assert the invariant cosine ≤ sqrt(coverage) for every spectrum. Tabulate per-spectrum coverage, cosine, and annotation pass/fail into casmi_coverage_cosine_metrics.csv, then plot joint distributions of coverage vs. cosine with marginal annotation rates and medians (expected ~50% annotation and ~80% median coverage for CASMI 2016 vs ~25% and ~32% for CASMI 2022, with ~62% of CASMI 16 spectra at cosine >0.7).

## Related tools

- **FIORA** (Graph neural network whose fiora-predict CLI generates the predicted mgf spectra whose cosine similarity and intensity coverage are computed and bound-checked) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i casmi16_input.csv -o casmi16_predicted.mgf --model default
```

## Evaluation signals

- Inequality cosine ≤ sqrt(annotated intensity coverage) holds numerically for every per-spectrum pair on both datasets
- Computed annotation rates fall near reported values: ~50% (CASMI 2016) and ~25% (CASMI 2022)
- Median intensity coverage matches reported ~80% (CASMI 2016) and ~32% (CASMI 2022)
- Roughly 62% of CASMI 2016 spectra achieve cosine similarity > 0.7
- Every retained spectrum passes hard filters: ≥2 peak matches, coverage ≥ 50%, precursor intensity ≤ 90%

## Limitations

- Not all collision energies for all CASMI 2022 challenge spectra were found during extraction, limiting annotation coverage
- FIORA performs only single-step fragmentation; the lack of multi-step fragmentation is currently the biggest limitation
- Cosine similarity is bounded by the square root of intensity coverage, so low annotation rates (as in CASMI 22 with 25% annotation, 32% median coverage) inherently cap achievable scores

## Evidence

- [discussion] Hard filters of at least two peak matches, 50% coverage and 90% precursor intensity: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] CASMI 2016 stepped collision energy prediction strategy: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single"
- [discussion] CASMI 2022 spectrum extraction parameters: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [discussion] Soft filtering criteria for spectra: "soft filters required that each spectrum have either 50% of the total peaks or at least five peaks matched by fragments, or an exceptionally high intensity coverage of 0.8."
- [readme] fiora-predict CLI usage: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
