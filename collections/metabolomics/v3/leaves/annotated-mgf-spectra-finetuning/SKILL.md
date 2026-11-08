---
name: annotated-mgf-spectra-finetuning
description: Use when a pretrained de novo sequencing model shows a systematic bias
  on a target spectral domain (e.g. reduced precision on non-tryptic peptides) and
  annotated MGF training/validation/test splits for that domain are available.
when_to_use_negative:
- The domain of interest is already well covered by the tryptic MassIVE-KB training
  distribution — the pretrained model's precision will not improve and fine-tuning
  risks overfitting a small PSM set.
- No annotated PSMs exist for the target domain — fine-tuning requires labeled peptide-spectrum
  matches in MGF (or convertible) form; unannotated spectra cannot drive supervised
  fine-tuning.
- You need a full de novo model from scratch with new PTMs or a new architecture rather
  than adaptation of the pretrained checkpoint.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer de novo sequencing model whose pretrained checkpoint is fine-tuned
    on annotated MGF PSMs and used for prediction/evaluation with beam search decoding
  repo: https://github.com/Noble-Lab/casanovo
- name: PDV
  role: Graphical interface for inspecting Casanovo outputs after fine-tuned prediction
  repo: https://github.com/wenbostar/PDV/
- name: limelight-import-casanovo
  role: Converter for generating Limelight XML from Casanovo results for downstream
    viewing
  repo: https://github.com/yeastrc/limelight-import-casanovo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:894fffa8107c463f0e618fafd1d59260406fd5a66d8e2984137e2ec752c4e032
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - PDV
  - limelight-import-casanovo
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Annotated MGF spectra fine-tuning

## Summary

Fine-tune a pretrained Casanovo transformer checkpoint on annotated MGF peptide-spectrum matches (e.g. a non-enzymatic dataset) to adapt de novo sequencing to spectral domains the base model handles poorly, such as non-tryptic or immunopeptidomics peptides.

## When to use

Use when a pretrained de novo sequencing model shows a systematic bias on a target spectral domain (e.g. reduced precision on non-tryptic peptides) and annotated MGF training/validation/test splits for that domain are available.

## When NOT to use

- The domain of interest is already well covered by the tryptic MassIVE-KB training distribution — the pretrained model's precision will not improve and fine-tuning risks overfitting a small PSM set.
- No annotated PSMs exist for the target domain — fine-tuning requires labeled peptide-spectrum matches in MGF (or convertible) form; unannotated spectra cannot drive supervised fine-tuning.
- You need a full de novo model from scratch with new PTMs or a new architecture rather than adaptation of the pretrained checkpoint.

## Inputs

- Annotated MGF training spectra file (training_spectra.mgf) of domain-specific PSMs
- Annotated MGF validation spectra file (validation_spectra.mgf)
- Annotated MGF test spectra split (e.g. 100,000-PSM non-enzymatic test set)
- Pretrained Casanovo checkpoint (pretrained.ckpt)

## Outputs

- Fine-tuned Casanovo checkpoint with lowest validation loss (<output_root>.best.ckpt)
- Peptide sequence predictions for the test split (mzTab export supported)
- Amino-acid and peptide-level precision evaluation metrics (e.g. average precision)
- C-terminal lysine/arginine frequency distribution of predicted peptides

## How to apply

Download the annotated MGF training, validation, and test splits for the target domain (e.g. the non-enzymatic dataset MassIVE MSV000094014). Instead of training from scratch, fine-tune an existing pretrained Casanovo checkpoint with `casanovo train --model pretrained.ckpt -p validation_spectra.mgf training_spectra.mgf`, letting periodic validation runs snapshot weights and keeping the checkpoint with the lowest validation loss as the best model. Then predict and evaluate (`casanovo sequence --evaluate`) with both the fine-tuned and the original model on the held-out test split using beam search decoding. The rationale is that the pretrained model encodes strong general spectral-to-peptide priors from 30 million MassIVE-KB PSMs, so fine-tuning on the smaller domain-specific PSM set corrects biases (like C-terminal lysine/arginine enrichment from tryptic training data) without needing the full 30-million-PSM training budget.

## Related tools

- **Casanovo** (Transformer de novo sequencing model whose pretrained checkpoint is fine-tuned on annotated MGF PSMs and used for prediction/evaluation with beam search decoding) — https://github.com/Noble-Lab/casanovo
- **PDV** (Graphical interface for inspecting Casanovo outputs after fine-tuned prediction) — https://github.com/wenbostar/PDV/
- **limelight-import-casanovo** (Converter for generating Limelight XML from Casanovo results for downstream viewing) — https://github.com/yeastrc/limelight-import-casanovo

## Examples

```
casanovo train --model pretrained.ckpt -p validation_spectra.mgf training_spectra.mgf && casanovo sequence --evaluate test_spectra.mgf
```

## Evaluation signals

- Peptide-level average precision on the held-out non-enzymatic test split should rise from ~0.60 (original Casanovo) to ~0.83 for the fine-tuned model.
- Amino-acid and peptide precision measures emitted by `casanovo sequence --evaluate` should both improve on the fine-tuned checkpoint.
- The checkpoint selected should be the one with the lowest validation loss across val_check_interval snapshots during fine-tuning.
- C-terminal lysine/arginine (K/R) frequency distribution of predicted peptides should be visibly reduced relative to the original model, matching Fig. 3a-b.
- Fine-tuned predictions on test spectra should remain valid amino-acid sequences consumable by downstream evaluation and mzTab export.

## Limitations

- Fine-tuning only corrects biases covered by the new training PSMs; performance on spectra outside both the pretraining and fine-tuning distributions is untested.
- The article notes the absence of a rigorous confidence-estimation method for de novo sequencing, so fine-tuned prediction scores cannot be interpreted as calibrated statistical confidences.
- No exhaustive benchmark of fine-tuning strategies for de novo sequencing tools exists; the field would benefit from rigorous comparisons of this growing set of tools.
- Model checkpoints depend on the val_check_interval snapshotting scheme; a suboptimal interval may miss the lowest-validation-loss weights.

## Evidence

- [intro] Fine-tune a non-enzymatic version of Casanovo for tasks such as immunopeptidomics: "Additionally, we fine-tune a non-enzymatic version of Casanovo for tasks such as immunopeptidomics."
- [methods] available as annotated MGF files on MassIVE with dataset identifier MSV000094014: "available as annotated MGF files on MassIVE with [dataset identifier MSV000094014]"
- [methods] you can fine-tune an existing pre-trained Casanovo model on data with new PTMs instead of training from scratch: "you can fine-tune an existing pre-trained Casanovo model on data with new PTMs instead of training from scratch"
- [intro] We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [readme] Casanovo is a state-of-the-art deep learning tool designed for de novo peptide sequencing: "Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing.
Powered by a transformer neural network, Casanovo "translates" peaks in MS/MS spectra into amino acid"
