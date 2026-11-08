---
name: casmi-benchmark-evaluation
description: Use when evaluating predicted tandem mass spectra against CASMI 2016/2022
  challenge data, or when you need to quantify how peak-annotation coverage bounds
  cosine similarity of in silico fragmentation predictions.
when_to_use_negative:
- The task is retraining or hyperparameter tuning of FIORA itself rather than benchmarking
  prediction quality on CASMI challenge data
- Comparing against CFM-ID, where evaluation must instead use the closest matching
  collision energy among 10, 20, 40 eV rather than a merged stepped spectrum
- Input spectra have missing collision energies or unclear formats, which the article
  explicitly excludes
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Graph neural network used via fiora-predict to generate predicted tandem MS/MS
    spectra for CASMI compounds at all 3 collision energies, merged into stepped spectra
  repo: https://github.com/BAMeScience/fiora
- name: fiora-predict
  role: CLI command (-i input csv -o predicted.mgf, --model default) that produces
    predicted spectra in mgf format for evaluation against observed challenge spectra
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: Baseline spectral prediction method from coleygroup/ms-pred against which
    FIORA's CASMI benchmark performance is compared; trained with all CASMI test compounds
    removed
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:d5cd510ab27a5d56a9a7ed91a36b37de284049226a7d1460df492999f09ed64e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  - ICEBERG
  - fiora-predict
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# casmi-benchmark-evaluation

## Summary

Benchmark FIORA's predicted MS/MS spectra against the CASMI 2016 and CASMI 2022 challenge sets by relating annotated peak-intensity coverage to spectral_sqrt_cosine similarity, reproducing Fig. 6. Coverage metrics are validated against reported values (~50%/25% annotation rates; ~80%/32% median coverage) and checked against the sqrt(coverage) >= cosine invariant.

## When to use

Use when evaluating predicted tandem mass spectra against CASMI 2016/2022 challenge data, or when you need to quantify how peak-annotation coverage bounds cosine similarity of in silico fragmentation predictions.

## When NOT to use

- The task is retraining or hyperparameter tuning of FIORA itself rather than benchmarking prediction quality on CASMI challenge data
- Comparing against CFM-ID, where evaluation must instead use the closest matching collision energy among 10, 20, 40 eV rather than a merged stepped spectrum
- Input spectra have missing collision energies or unclear formats, which the article explicitly excludes

## Inputs

- CASMI 2016 challenge input CSV with columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' (stepped collision energies)
- CASMI 2022 challenge input CSV and consensus spectra extracted with 10 ppm precursor tolerance and 5 s RT window
- Observed CASMI challenge spectra (mgf) downloaded from the CASMI contest webpage
- FIORA default model checkpoint packaged under fiora/resources/models

## Outputs

- casmi_coverage_cosine_metrics.csv with per-spectrum intensity coverage, cosine similarity, and annotation pass/fail per dataset
- fig6_coverage_cosine_joint_distribution.png showing joint distributions of coverage vs cosine for CASMI 2016 and CASMI 2022 with marginal annotation rates and medians
- Predicted spectra in mgf format (one merged 'stepped' spectrum per CASMI 2016 compound)

## How to apply

Prepare input CSVs for both CASMI sets with headers 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type'; for CASMI 2016 use the stepped collision energies (predict at all 3 CEs with fiora-predict and merge into a single 'stepped' spectrum), and for CASMI 2022 use consensus spectra extracted with OpenMS at 10 ppm precursor tolerance and a 5 s retention time window. Run fiora-predict with the default model to generate predicted spectra in mgf format. Match predicted fragments to observed peaks and enforce the paper's hard filters: at least two peak matches, minimum intensity coverage of 50%, and maximum precursor intensity of 90%. Compute spectral_sqrt_cosine per spectrum using FIORA's evaluation scoring and numerically verify cosine never exceeds the square root of intensity coverage. Tabulate coverage, cosine, and pass/fail per dataset, then plot joint distributions with marginals to reproduce Fig. 6; validate that annotation rates (~50% CASMI 2016 vs ~25% CASMI 2022) and median coverages (~80% vs ~32%) fall near the reported values.

## Related tools

- **FIORA** (Graph neural network used via fiora-predict to generate predicted tandem MS/MS spectra for CASMI compounds at all 3 collision energies, merged into stepped spectra) — https://github.com/BAMeScience/fiora
- **fiora-predict** (CLI command (-i input csv -o predicted.mgf, --model default) that produces predicted spectra in mgf format for evaluation against observed challenge spectra) — https://github.com/BAMeScience/fiora
- **ICEBERG** (Baseline spectral prediction method from coleygroup/ms-pred against which FIORA's CASMI benchmark performance is compared; trained with all CASMI test compounds removed) — https://github.com/samgoldman97/ms-pred

## Examples

```
fiora-predict -i casmi16_input.csv -o casmi16_predicted.mgf --model default
```

## Evaluation signals

- Per-spectrum spectral_sqrt_cosine value never exceeds the square root of its annotated intensity coverage for every spectrum in both datasets
- CASMI 2016 annotation rate is near 50% and CASMI 2022 near 25% after applying the hard filters
- Median annotated peak-intensity coverage is near 80% for CASMI 2016 and near 32% for CASMI 2022
- Every evaluated spectrum passes the hard filters: at least two peak matches, minimum intensity coverage of 50%, maximum precursor intensity of 90%
- fiora-predict output matches the expected reference results in examples/expected_output.mgf (automatically checked by pytest)

## Limitations

- Not all collision energies for all CASMI 22 challenge spectra were found during extraction, reducing coverage of the 2022 benchmark
- Multiple measurements at the same collision energy must be merged into consensus spectra for CASMI 22, which can obscure per-spectrum differences
- The 50% minimum intensity coverage filter excludes spectra with low annotated coverage, which are precisely the low-quality cases of interest for the coverage-cosine relationship

## Evidence

- [discussion] CASMI 16 stepped CE: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single “stepped" spectrum."
- [discussion] CASMI 22 extraction: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [discussion] hard filters: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [readme] input format: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type" and rows listing individual queries."
- [readme] CLI usage: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
- [discussion] training splits: "To preserve the integrity of the test results, all compounds from the test sets were removed from the training data for FIORA and ICEBERG."
