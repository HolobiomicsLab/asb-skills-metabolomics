---
name: held-out-lipid-species-validation-splitting
description: Use when judging whether a trained lipid MS2 annotation model learned
  transferable fragmentation patterns — i.e., testing on spectra of lipid species
  completely held out from training rather than re-scoring training or random validation
  spectra.
when_to_use_negative:
- The evaluation set contains spectra of lipid species that were present in the training
  data — a standard validation accuracy, not a generalization test, would be measured
  instead.
- Inputs are raw profile spectra or mzML prediction files rather than the preprocessed
  HDF5 spectra with a defined held-out split YAML.
- The spectra are blank measurements with no target lipid species present, where the
  model should simply predict the end-of-sequence token rather than a lipid identity.
edam_operation: http://edamontology.org/operation_0282
tools:
- name: LipiDetective
  role: Transformer model whose pre-trained weights are run with greedy and beam search
    decoding to identify unseen lipid species in MS2 spectra
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Framework implementing the LipiDetective transformer module used for inference
    on held-out spectra
- name: PyTorch Lightning
  role: Used for multi-GPU training of the model whose weights are loaded for this
    evaluation
- name: h5py
  role: Library used to load the preprocessed MS2 spectra from the HDF5 file for validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:ed2822b7d214bbbc91a7a8d91c18867672a110d20825d11c6c55b3bda39cfc6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - h5py
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# held-out-lipid-species-validation-splitting

## Summary

Evaluates whether LipiDetective, a transformer for identifying molecular lipid species from MS2 tandem mass spectra, generalizes to lipid species for which it has never seen a spectrum, using held-out validation splits and strict token-level accuracy scoring.

## When to use

Use when judging whether a trained lipid MS2 annotation model learned transferable fragmentation patterns — i.e., testing on spectra of lipid species completely held out from training rather than re-scoring training or random validation spectra.

## When NOT to use

- The evaluation set contains spectra of lipid species that were present in the training data — a standard validation accuracy, not a generalization test, would be measured instead.
- Inputs are raw profile spectra or mzML prediction files rather than the preprocessed HDF5 spectra with a defined held-out split YAML.
- The spectra are blank measurements with no target lipid species present, where the model should simply predict the end-of-sequence token rather than a lipid identity.

## Inputs

- Preprocessed MS2 tandem mass spectra stored in an HDF5 file (smoothed, baseline-corrected, peak-picked, base-peak-normalized, trimmed to m/z 50-1600)
- Validation-split YAML defining which lipid species are held out from training
- Pre-trained LipiDetective transformer weights (PyTorch/PyTorch Lightning checkpoint)
- Training configuration YAML (paths to HDF5 files and model parameters, per README template config/config_templates)

## Outputs

- Per-spectrum predictions CSV (greedy and beam search decoder outputs with labels)
- Per-epoch metrics CSV in the experiment output directory
- Total prediction accuracy and lipid component accuracy on the unseen-species validation split (e.g. PG 67.3% total accuracy)

## How to apply

Load the preprocessed MS2 spectra from the HDF5 file together with the validation-split YAML that defines which lipid species were held out from training. Load the pre-trained LipiDetective transformer weights (PyTorch, trained via PyTorch Lightning) using the training configuration YAML. Run inference with both greedy and beam search decoders on the validation spectra of unseen lipid species. Score each prediction with total prediction accuracy (a prediction counts correct only when every token — headgroup, fatty acid chains, adduct — matches the label) and lipid component accuracy (correctly predicted tokens / total tokens). Write per-spectrum predictions and per-epoch metrics to CSV files in the experiment output directory; expect the best unseen-species performance on PG, at 67.3% total prediction accuracy.

## Related tools

- **LipiDetective** (Transformer model whose pre-trained weights are run with greedy and beam search decoding to identify unseen lipid species in MS2 spectra) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Framework implementing the LipiDetective transformer module used for inference on held-out spectra)
- **PyTorch Lightning** (Used for multi-GPU training of the model whose weights are loaded for this evaluation)
- **h5py** (Library used to load the preprocessed MS2 spectra from the HDF5 file for validation)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- A prediction is counted correct only when every token (headgroup, fatty acid chains, adduct) matches the label — no partial matches should be tallied as correct.
- Both accuracy measures are reported on the unseen-species set: total prediction accuracy and lipid component accuracy (correctly predicted tokens / total tokens).
- Best-performing lipid class on the held-out split should be around PG at 67.3% total prediction accuracy.
- Per-spectrum predictions CSV and per-epoch metrics CSV exist in the experiment output directory after the run.
- Greedy and beam search decoder outputs are both generated and compared on the same validation spectra.

## Limitations

- Accuracy varies by lipid class; the reported best unseen-species performance (67.3% total accuracy on PG) is still well below perfect identification.
- The model was trained on a relatively small dataset of 268,720 spectra, so generalization depends on the learned fragmentation patterns rather than memorization.
- Only the HDF5 prediction path supports this split-based evaluation; mzML input in the CLI targets prediction on new files, not the held-out species protocol.

## Evidence

- [abstract] abstract claim of generalization: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
- [other] task finding on accuracy: [evidence span withheld — anchor status: not_found]
- [methods] transformer implementation: "LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]"
- [readme] README feature description: [evidence span withheld — anchor status: external_doc]
