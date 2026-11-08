---
name: parity-plot-visualization
description: Use when a trained FIORA model checkpoint and annotated compound sets
  (RT and CCS labels) are available and you need to quantify/visualize regression
  accuracy of RT/CCS predictions against simple baselines on held-out splits.
when_to_use_negative:
- Using NIST'17 as the annotation source for this task — it does not provide RT and
  CCS values of the measured compounds
- Evaluating spectral (peak-intensity) prediction quality rather than the auxiliary
  scalar properties RT and CCS
- Applying to compounds outside the trained ionization modes ([M+H]+ and [M-H]-) or
  outside the dataset's weight/energy constraints (max 1000 Da, max 100 eV)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Trains RT and CCS prediction submodules (fiora-train) and generates RT/CCS
    predictions (fiora-predict with --rt and --ccs flags) for parity-plot evaluation
  repo: https://github.com/BAMeScience/fiora
- name: MS-DIAL spectral library
  role: Public .msp library providing the annotated MS/MS spectra from which the 409-compound
    RT and 1346-compound CCS subsets are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:13a2c4ca297b251dd91c0cec2360a8cd2a361b9606223975b5bed5386b5dab08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  - MS-DIAL spectral library
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# parity-plot-visualization

## Summary

Reproduce predicted-versus-observed parity plots for FIORA's auxiliary property predictions — retention time (RT) and collision cross section (CCS) — annotated with R² values against baseline linear regressions (Fig. 5 of the FIORA paper). This validates that FIORA's modular graph architecture generalizes beyond spectra to orthogonal identification dimensions.

## When to use

Use when a trained FIORA model checkpoint and annotated compound sets (RT and CCS labels) are available and you need to quantify/visualize regression accuracy of RT/CCS predictions against simple baselines on held-out splits.

## When NOT to use

- Using NIST'17 as the annotation source for this task — it does not provide RT and CCS values of the measured compounds
- Evaluating spectral (peak-intensity) prediction quality rather than the auxiliary scalar properties RT and CCS
- Applying to compounds outside the trained ionization modes ([M+H]+ and [M-H]-) or outside the dataset's weight/energy constraints (max 1000 Da, max 100 eV)

## Inputs

- MS-DIAL spectral library (.msp) with RT and CCS annotations
- FIORA model checkpoint (e.g. checkpoints/fiora.pt) trained with RT and CCS submodules
- CASMI 2016 challenge compound set
- Input CSV with columns Name, SMILES, Precursor_type, CE, Instrument_type for fiora-predict

## Outputs

- RT and CCS predictions for validation/test and CASMI 16 compounds
- R² values for FIORA versus baseline regressions on each split
- Predicted-vs-observed parity plots for RT and CCS annotated with R² values (reproducing Fig. 5)

## How to apply

Extract compounds with RT and CCS annotations from the MS-DIAL spectral library (.msp format), yielding a 409-compound RT subset and a 1346-compound CCS subset, split 80% training with the remaining 20% split evenly between validation and testing. Train the RT and CCS submodules with fiora-train (200 epochs, ADAM optimizer, weighted MSE loss, plateau-reducing learning-rate scheduler on validation loss), saving the checkpoint (e.g. checkpoints/fiora.pt) and optionally per-epoch history via --history-out. Run fiora-predict with --rt and --ccs enabled on validation/test compounds and CASMI 2016 compounds. Fit baselines on the same splits: logP-based linear regression for RT and precursor-m/z-based linear regression for CCS. Compute R² per split (targets: RT R² ≈ 0.70; CCS R² ≈ 0.93 on the test split, ≈ 0.86 on CASMI 16, versus 0.9 and 0.80 for the m/z baseline) and render predicted-vs-observed parity plots annotated with these R² values.

## Related tools

- **FIORA** (Trains RT and CCS prediction submodules (fiora-train) and generates RT/CCS predictions (fiora-predict with --rt and --ccs flags) for parity-plot evaluation) — https://github.com/BAMeScience/fiora
- **MS-DIAL spectral library** (Public .msp library providing the annotated MS/MS spectra from which the 409-compound RT and 1346-compound CCS subsets are extracted)

## Examples

```
fiora-predict -i test_compounds.csv -o predictions.mgf --model checkpoints/fiora.pt --rt --ccs
```

## Evaluation signals

- CCS test-split R² ≈ 0.93 for FIORA versus ≈ 0.9 for the precursor-m/z linear regression baseline
- CCS R² on CASMI 16 compounds ≈ 0.86 for FIORA versus 0.80 for the m/z baseline
- RT R² ≈ 0.70 against the logP-based linear regression baseline
- Parity plot points fall close to the identity (y = x) line for both RT and CCS panels
- Training converges within 200 epochs with validation loss plateau triggering the learning-rate scheduler (verifiable via --history-out JSON/CSV)

## Limitations

- Hyperparameters were not specifically tuned for the RT and CCS prediction task; the feature is still in prototype stage
- RT and CCS subsets are small (409 RT compounds; 1346 CCS compounds), so R² estimates may be sensitive to the 80/10/10 split
- Exact model specification is only given in the Supplementary Information, not the main text

## Evidence

- [results] FIORA's CCS predictions outperform a linear regression based on precursor m/z, achieving R2 values of 0.93 on the test split (vs 0.9) and 0.86 for CASMI 16 compounds (vs 0.80).: [evidence span withheld — anchor status: not_found]
- [discussion] FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing.: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [discussion] The NIST'17 MS/MS library does not provide information about RT and CCS values of the measured compounds.: "The NIST’17 MS/MS library does not provide information about RT and CCS values of the measured compounds."
- [discussion] The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss.: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces"
- [abstract] also facilitates the prediction of additional features, such as retention time and collision cross section: "also facilitates the prediction of additional features, such as retention time and collision cross section"
- [readme] fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs]"
