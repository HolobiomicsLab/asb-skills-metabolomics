---
name: transformer-de-novo-sequencing-model-training
description: Use when annotated PSM training data (e.g. MGF files from MassIVE) are
  available and a pretrained Casanovo checkpoint must be adapted to a new peptide
  regime, such as non-enzymatic or PTM-containing spectra.
when_to_use_negative:
- The task is a standard tryptic database-search identification where the protein
  set is known in advance — SEQUEST-style search engines are the conventional choice
- Input spectra are raw/unconverted vendor formats rather than Casanovo-compatible
  annotated or msconvert-processed files
- You need calibrated confidence scores for de novo predictions — the article notes
  the field lacks a rigorous confidence estimation method
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer model trained from scratch or fine-tuned from a pretrained checkpoint;
    provides casanovo train and casanovo sequence --evaluate with beam search decoding
  repo: https://github.com/Noble-Lab/casanovo
- name: msconvert
  role: Converts input raw data files to a Casanovo-compatible format before training
    or prediction
- name: PDV
  role: Graphical inspection of Casanovo outputs during result review
  repo: https://github.com/wenbostar/PDV/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:65b25f2661c9a08b1799d7312265c6cd73848839c7e8136ba93c911f301dfa69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - PDV
  - msconvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# transformer-de-novo-sequencing-model-training

## Summary

Train or fine-tune a transformer-based de novo peptide sequencing model (Casanovo) from annotated tandem MS/MS spectra, translating peak sequences into amino acid sequences. Fine-tuning the pretrained checkpoint on non-enzymatic PSMs (Casanovone) reduces tryptic bias for immunopeptidomics-style applications.

## When to use

Use when annotated PSM training data (e.g. MGF files from MassIVE) are available and a pretrained Casanovo checkpoint must be adapted to a new peptide regime, such as non-enzymatic or PTM-containing spectra.

## When NOT to use

- The task is a standard tryptic database-search identification where the protein set is known in advance — SEQUEST-style search engines are the conventional choice
- Input spectra are raw/unconverted vendor formats rather than Casanovo-compatible annotated or msconvert-processed files
- You need calibrated confidence scores for de novo predictions — the article notes the field lacks a rigorous confidence estimation method

## Inputs

- annotated MGF training, validation, and test split files (e.g. from MassIVE MSV000094014)
- pretrained Casanovo checkpoint (pretrained.ckpt)
- optional Casanovo YAML configuration file (-p)
- MassIVE-KB v1 30 million PSM training/validation/test splits (Zenodo 10.5281/zenodo.14967861) for training from scratch

## Outputs

- fine-tuned model checkpoint with lowest validation loss (<output_root>.best.ckpt)
- predicted peptide sequences with amino-acid and peptide precision evaluation metrics
- mzTab export of spectrum predictions
- C-terminal K/R frequency distribution summary of predicted peptides

## How to apply

Reframe the task as machine translation: MS/MS peak sequences are the source language and amino acid sequences the target, using the Casanovo transformer trained on 30 million confident MassIVE-KB PSMs. For fine-tuning, download annotated MGF training/validation/test splits (e.g. the non-enzymatic dataset MassIVE MSV000094014 / doi:10.25345/C5KS6JG0W) and run 'casanovo train --model pretrained.ckpt -p validation_spectra.mgf training_spectra.mgf', letting periodic validation snapshots record weights and keeping the checkpoint with the lowest validation loss as <output_root>.best.ckpt. Predict with 'casanovo sequence --evaluate' using beam search decoding on the held-out test split (e.g. 100,000 PSMs), then tabulate amino-acid and peptide precision. Judge success by comparing against the original pretrained model and by inspecting the C-terminal lysine/arginine frequency distribution of predictions, which should shift away from tryptic K/R ends.

## Related tools

- **Casanovo** (Transformer model trained from scratch or fine-tuned from a pretrained checkpoint; provides casanovo train and casanovo sequence --evaluate with beam search decoding) — https://github.com/Noble-Lab/casanovo
- **msconvert** (Converts input raw data files to a Casanovo-compatible format before training or prediction)
- **PDV** (Graphical inspection of Casanovo outputs during result review) — https://github.com/wenbostar/PDV/

## Examples

```
casanovo train --model pretrained.ckpt -p validation_spectra.mgf training_spectra.mgf; casanovo sequence --evaluate
```

## Evaluation signals

- Fine-tuned model reaches peptide-level average precision ≈ 0.83 versus ≈ 0.60 for the original pretrained Casanovo on the 100,000-PSM non-enzymatic test split
- C-terminal lysine/arginine frequency distribution of predicted peptides is visibly reduced relative to the tryptic-biased original model
- Selected checkpoint corresponds to the lowest validation loss among periodic validation snapshots during fine-tuning
- Both amino-acid and peptide precision measures emitted during 'casanovo sequence --evaluate' are tabulated for each model
- Training was seeded from the 30 million confident MassIVE-KB PSMs that the article reports as particularly good training data

## Limitations

- Original Casanovo carries a tryptic bias from MassIVE-KB training data, degrading non-enzymatic peptide prediction until fine-tuned
- Absence of a rigorous confidence estimation method for de novo sequencing remains a key bottleneck
- The impact of the number of model parameters on de novo sequencing performance is an open question left for future work
- Fine-tuning is suggested for new PTMs as an alternative to training from scratch, but from-scratch training requires the massive MassIVE-KB collection

## Evidence

- [abstract] We also develop a version of Casanovo that is fine-tuned for non-enzymatic peptides: "We also develop a version of Casanovo that is fine-tuned for non-enzymatic peptides"
- [methods] fine-tune on data with new PTMs: "you can fine-tune an existing pre-trained Casanovo model on data with new PTMs instead of training from scratch"
- [methods] annotated MGF files on MassIVE: "available as annotated MGF files on MassIVE with [dataset identifier MSV000094014]"
- [intro] beam search decoding: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [readme] README: transformer translates spectra to peptides: "Powered by a transformer neural network, Casanovo "translates" peaks in MS/MS spectra into amino acid sequences with remarkable precision."
