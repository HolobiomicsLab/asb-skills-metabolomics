---
name: retention-time-prediction-modeling
description: Use when you need to predict retention times for compounds from SMILES/structures
  to complement MS/MS spectral matching, or to benchmark a learned RT predictor against
  a logP-based linear regression baseline.
when_to_use_negative:
- Your reference library is NIST'17 — it does not provide RT (or CCS) values for measured
  compounds, so no RT training labels are available
- You need spectrally annotated RT on chromatographically incompatible setups — the
  RT model's hyperparameters were not tuned for this task and the feature is prototype-stage
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Graph neural network whose RT submodule is trained via fiora-train and applied
    via fiora-predict --rt to predict retention times from structure
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:396dcb2d396ea42a7d79ca21e32ddb318e2b06d2d921bfd8f248b075711d94e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# retention-time-prediction-modeling

## Summary

Train and apply FIORA's RT prediction submodule to estimate LC retention times from molecular structure, alongside CCS prediction, as additional dimensions for MS-based compound identification beyond spectral similarity.

## When to use

Use when you need to predict retention times for compounds from SMILES/structures to complement MS/MS spectral matching, or to benchmark a learned RT predictor against a logP-based linear regression baseline.

## When NOT to use

- Your reference library is NIST'17 — it does not provide RT (or CCS) values for measured compounds, so no RT training labels are available
- You need spectrally annotated RT on chromatographically incompatible setups — the RT model's hyperparameters were not tuned for this task and the feature is prototype-stage

## Inputs

- MS-DIAL spectral library in .msp format with RT annotations (~409 compounds with RT values)
- CSV input for fiora-predict with columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type'

## Outputs

- Trained FIORA RT model checkpoint (e.g. checkpoints/fiora.pt)
- Per-epoch training history (. or .csv via --history-out)
- Predicted RT values for validation/test compounds
- Parity plot of predicted-vs-observed RT annotated with R2 (reproducing Fig. 5)

## How to apply

Extract compounds with RT annotations from the MS-DIAL spectral library in .msp format (forming the 409-compound RT subset), filtering compounds with SMILES/MOL/InChiKey inconsistencies, and split 80% for training with the remaining 20% split evenly between validation and testing. Train the FIORA RT submodule using the fiora-train CLI with 200 epochs, ADAM optimizer, weighted MSE loss, and a plateau-reducing learning-rate scheduler on validation loss, saving the checkpoint (e.g. checkpoints/fiora.pt) and optionally per-epoch history via --history-out. Run fiora-predict with --rt enabled to generate RT predictions for held-out compounds. Fit a logP-based linear regression on the same training split as the baseline comparator. Judge success by parity of predicted-vs-observed RT with target R2 ≈ 0.70 on the test split.

## Related tools

- **FIORA** (Graph neural network whose RT submodule is trained via fiora-train and applied via fiora-predict --rt to predict retention times from structure) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i examples/example_input.csv -o predictions.mgf --rt --model checkpoints/fiora.pt
```

## Evaluation signals

- RT prediction reaches R2 ≈ 0.70 on the held-out test split
- Learned RT model outperforms the logP-based linear regression fitted on the same training split
- Predicted-vs-observed parity plot shows unbiased scatter around the diagonal without systematic bias
- Validation loss plateaus under the plateau-reducing learning-rate scheduler within the 200-epoch training run
- Predictions complete successfully with --rt enabled and --no-rt disabled flag states verified in fiora-predict output

## Limitations

- Hyperparameters were not specifically tuned for the RT prediction task; the feature is still in prototype stage
- Only 409 compounds carry RT annotations in the MS-DIAL library, limiting training data
- NIST'17, the largest authentic-standard library, provides no RT or CCS values, so it cannot supplement the training set

## Evidence

- [discussion] The NIST’17 MS/MS library does not provide information about RT and CCS values of the measured compounds.: "The NIST’17 MS/MS library does not provide information about RT and CCS values of the measured compounds."
- [discussion] Hyperparameters were not specifically tuned for this task... While this feature is still in its prototype stage: [evidence span withheld — anchor status: not_found]
- [discussion] The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss.: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [discussion] we process a public library available in .msp format on the MS-DIAL website36, which contains a collection of annotated MS/MS spectra: "we process a public library available in .msp format on the MS-DIAL website36, which contains a collection of annotated MS/MS spectra"
- [readme] Use spectral prediction function as follows: fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
