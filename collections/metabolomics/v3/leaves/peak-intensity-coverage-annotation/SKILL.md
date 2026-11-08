---
name: peak-intensity-coverage-annotation
description: Use when evaluating FIORA (or similar) spectral predictions against CASMI
  challenge data and you need to quantify annotated intensity coverage, enforce the
  paper's annotation filters, and verify cosine ≤ sqrt(coverage) per spectrum.
when_to_use_negative:
- The goal is compound identification / candidate ranking rather than quantifying
  spectral prediction quality versus annotation coverage
- Observed spectra lack collision-energy annotations or have unclear formats, so the
  paper's matched-CE evaluation and coverage filters cannot be applied
- Evaluating CFM-ID predictions, which require evaluation at the closest matching
  collision energy among 10, 20, 40 eV instead of merged stepped spectra
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Graph neural network used via fiora-predict to generate predicted MS/MS spectra
    in mgf format at the required collision energies for coverage and cosine computation
  repo: https://github.com/BAMeScience/fiora
- name: fiora-predict
  role: CLI invoked as fiora-predict -i <casmi_input.csv> -o <predicted.mgf> --model
    default to produce the predicted spectra matched against observed peaks
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:fbf7d4e626b422439fcc7d066a18ec8a37b8f27e53aec334f00f272887901b54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  - fiora-predict
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-intensity-coverage-annotation

## Summary

Compute per-spectrum annotated peak-intensity coverage of FIORA-predicted MS/MS spectra against observed CASMI 2016/2022 challenge spectra and relate it to square-root cosine similarity, revealing that spectral prediction quality is bounded by sqrt(coverage).

## When to use

Use when evaluating FIORA (or similar) spectral predictions against CASMI challenge data and you need to quantify annotated intensity coverage, enforce the paper's annotation filters, and verify cosine ≤ sqrt(coverage) per spectrum.

## When NOT to use

- The goal is compound identification / candidate ranking rather than quantifying spectral prediction quality versus annotation coverage
- Observed spectra lack collision-energy annotations or have unclear formats, so the paper's matched-CE evaluation and coverage filters cannot be applied
- Evaluating CFM-ID predictions, which require evaluation at the closest matching collision energy among 10, 20, 40 eV instead of merged stepped spectra

## Inputs

- CASMI 2016 challenge input CSV with columns Name, SMILES, Precursor_type, CE (stepped collision energies), Instrument_type
- CASMI 2022 challenge consensus spectra extracted with 10 ppm precursor tolerance and 5 s retention time window
- Observed MS/MS spectra (mgf) for the CASMI 2016 and CASMI 2022 challenge sets

## Outputs

- casmi_coverage_cosine_metrics.csv with per-spectrum coverage, cosine similarity, and annotation pass/fail per dataset
- fig6_coverage_cosine_joint_distribution.png: joint distributions of intensity coverage vs cosine similarity for CASMI 2016 and CASMI 2022 with marginal annotation rates and medians
- Predicted spectra in mgf format (e.g., predicted.mgf) from fiora-predict

## How to apply

Prepare input CSVs with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' for both CASMI sets (stepped collision energies for CASMI 2016; consensus spectra extracted with 10 ppm precursor tolerance and 5 s RT window for CASMI 2022), then run fiora-predict with --model default to output mgf spectra, predicting at all 3 collision energies for CASMI 2016 and merging into a single 'stepped' spectrum. Match predicted fragment peaks to observed peaks and compute per-spectrum annotated peak-intensity coverage, enforcing the paper's hard filters: at least two peak matches, minimum intensity coverage of 50%, and maximum precursor intensity of 90%. Compute the square-root cosine similarity (spectral_sqrt_cosine) with the FIORA evaluation scoring and numerically verify cosine ≤ sqrt(coverage) for every spectrum. Tabulate coverage, cosine, and annotation pass/fail into a CSV and plot joint distributions with marginal annotation rates and medians (expected ~50% annotation and ~80% median coverage for CASMI 2016 vs ~25% and ~32% for CASMI 2022), reproducing Fig. 6. The rationale is that cosine similarity of a predicted spectrum cannot exceed the square root of the fraction of observed intensity its fragments annotate, so low annotation coverage caps achievable prediction quality.

## Related tools

- **FIORA** (Graph neural network used via fiora-predict to generate predicted MS/MS spectra in mgf format at the required collision energies for coverage and cosine computation) — https://github.com/BAMeScience/fiora
- **fiora-predict** (CLI invoked as fiora-predict -i <casmi_input.csv> -o <predicted.mgf> --model default to produce the predicted spectra matched against observed peaks) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i casmi16_input.csv -o casmi16_predicted.mgf --model default
```

## Evaluation signals

- Per-spectrum invariant: cosine similarity never exceeds the square root of annotated intensity coverage (sqrt(coverage) >= cosine) for every spectrum in both datasets
- CASMI 2016 annotation rate lands near ~50% with median coverage ~80%, and ~62% of spectra achieve cosine > 0.7
- CASMI 2022 annotation rate lands near ~25% with median coverage ~32%
- All spectra in the metrics CSV satisfy the hard filters: at least two peak matches, minimum intensity coverage of 50%, maximum precursor intensity of 90%
- Output CSV contains columns for per-spectrum coverage, cosine similarity, and annotation pass/fail for both CASMI 2016 and CASMI 2022

## Limitations

- Not all collision energies for all CASMI 2022 challenge spectra were found during extraction, reducing achievable coverage
- CASMI 2022's lower peak annotation rate (~25%) and median coverage (~32%) inherently cap cosine similarity, so low scores there reflect coverage, not only model quality
- FIORA's single-step fragmentation is its biggest limitation; multi-step fragmentation is not implemented, which constrains matched-fragment coverage for complex spectra

## Evidence

- [discussion] Stepped CE prediction for CASMI 2016: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them"
- [discussion] CASMI 22 extraction parameters: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [discussion] Hard annotation filters: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] Soft filters for annotation: "soft filters required that each spectrum have either 50% of the total peaks or at least five peaks matched by fragments, or an exceptionally high intensity coverage of 0.8."
- [readme] fiora-predict CLI usage: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
- [readme] Input CSV schema: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [discussion] Missing CASMI 22 collision energies: "Despite our efforts, not all collision energies for all challenge spectra were found."
