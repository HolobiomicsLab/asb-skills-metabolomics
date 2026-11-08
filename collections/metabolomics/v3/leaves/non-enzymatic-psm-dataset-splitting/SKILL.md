---
name: non-enzymatic-psm-dataset-splitting
description: Use when fine-tuning or benchmarking a de novo sequencing model on non-tryptic/non-enzymatic
  spectra, and you need annotated MGF training/validation/test splits rather than
  tryptic MassIVE-KB data.
when_to_use_negative:
- The task is standard tryptic DDA database searching or sequencing where MassIVE-KB-derived
  tryptic training data are appropriate — fine-tuning on non-enzymatic data is unnecessary
  and may reduce tryptic performance.
- Only a pretrained Casanovo model is needed for inference and no model retraining
  or non-tryptic benchmarking is planned — no split files need to be involved.
- The input spectra are DIA rather than DDA bottom-up tandem MS data.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Fine-tuned on the non-enzymatic training/validation MGF splits and used to
    sequence the held-out test split with beam search decoding and --evaluate metrics
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:64def513063235715dba7785bd4e8d0bf6ba9e5925f2106beb86d795cb62bf60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# non-enzymatic-psm-dataset-splitting

## Summary

Split non-enzymatic PSM data into training, validation, and test sets (delivered as annotated MGF files on MassIVE, MSV000094014) so that a pretrained Casanovo model can be fine-tuned and fairly evaluated on non-tryptic peptides.

## When to use

Use when fine-tuning or benchmarking a de novo sequencing model on non-tryptic/non-enzymatic spectra, and you need annotated MGF training/validation/test splits rather than tryptic MassIVE-KB data.

## When NOT to use

- The task is standard tryptic DDA database searching or sequencing where MassIVE-KB-derived tryptic training data are appropriate — fine-tuning on non-enzymatic data is unnecessary and may reduce tryptic performance.
- Only a pretrained Casanovo model is needed for inference and no model retraining or non-tryptic benchmarking is planned — no split files need to be involved.
- The input spectra are DIA rather than DDA bottom-up tandem MS data.

## Inputs

- Annotated MGF files for the non-enzymatic dataset (training, validation, and test splits) from MassIVE MSV000094014
- Pretrained Casanovo checkpoint (pretrained.ckpt)
- MassIVE-KB v1 30 million PSMs and its training/validation/test splits (Zenodo 10.5281/zenodo.14973856 and 10.5281/zenodo.14967861) for the original model reference

## Outputs

- Fine-tuned Casanovo checkpoint (<output_root>.best.ckpt) selected by lowest validation loss
- Peptide predictions and evaluation metrics (amino-acid and peptide precision, peptide-level average precision ~0.83 vs ~0.60 for the original model) on the 100,000-PSM non-enzymatic test split
- C-terminal lysine/arginine frequency distribution of predicted peptides for both models

## How to apply

Download the annotated MGF training, validation, and test splits for the non-enzymatic dataset from MassIVE (MSV000094014, doi:10.25345/C5KS6JG0W). Use the training and validation PSMs as fine-tuning inputs to the pretrained Casanovo checkpoint (`casanovo train --model pretrained.ckpt -p validation_spectra.mgf training_spectra.mgf`), reserving the ~100,000-PSM non-enzymatic test split exclusively for evaluation via `casanovo sequence --evaluate` with beam search decoding. The rationale is that Casanovo's original training corpus (30 million confident PSMs from MassIVE-KB) is largely tryptic, so a held-out non-enzymatic test set is required to measure reduction of tryptic bias without leakage. Keep the checkpoint with the lowest validation loss (<output_root>.best.ckpt) as the fine-tuned model, and assess K/R C-terminal frequencies of predictions alongside peptide-level average precision.

## Related tools

- **Casanovo** (Fine-tuned on the non-enzymatic training/validation MGF splits and used to sequence the held-out test split with beam search decoding and --evaluate metrics) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo train --model pretrained.ckpt -p validation_spectra.mgf training_spectra.mgf
```

## Evaluation signals

- Fine-tuned model reaches peptide-level average precision ≈0.83 versus ≈0.60 for the original Casanovo model on the 100,000-PSM non-enzymatic test split.
- C-terminal K/R frequencies of predicted peptides are visibly reduced after fine-tuning (reproducing Fig. 3a-b).
- Validation loss decreases during fine-tuning and the selected checkpoint (<output_root>.best.ckpt) is the one with the lowest validation loss.
- Test-split PSMs were never used for training or checkpoint selection (no leakage between the MGF splits).
- Amino-acid and peptide precision measures emitted during `casanovo sequence --evaluate` are tabulated for both models under identical decoding settings.

## Limitations

- The non-enzymatic dataset is specific to immunopeptidomics-style non-tryptic PSMs; performance gains may not transfer to enzymatic datasets trained from the same checkpoint.
- The field lacks a rigorous confidence-estimation method for de novo sequencing, so peptide-level average precision is the primary quality signal.
- Whether Casanovo outperforms standard database search in statistical power for a given DDA dataset remains an open question, so splits should not be presented as evidence of superior end-to-end identification power.

## Evidence

- [methods] Download the annotated MGF training, validation, and test splits for the non-enzymatic dataset from MassIVE (MSV000094014 / doi:10.25345/C5KS6JG0W).: "available as annotated MGF files on MassIVE with [dataset identifier MSV000094014]"
- [methods] Fine-tune the pretrained Casanovo checkpoint on the non-enzymatic training PSMs, keeping the checkpoint with the lowest validation loss.: "you can fine-tune an existing pre-trained Casanovo model on data with new PTMs instead of training from scratch"
- [intro] Additionally, we fine-tune a non-enzymatic version of Casanovo for tasks such as immunopeptidomics.: "Additionally, we fine-tune a non-enzymatic version of Casanovo for tasks such as immunopeptidomics."
- [discussion] Our experiments suggest that the carefully curated MassIVE-KB collection provides particularly good training data.: "Our experiments suggest that the carefully curated MassIVE-KB collection provides particularly good training data."
- [readme] Casanovo is particularly useful for immunopeptidomics, metaproteomics, paleoproteomics, venomics, or any setting in which you are interested in identifying peptides that may not be in your protein database.: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset, and is particularly useful for immunopeptidomics"
