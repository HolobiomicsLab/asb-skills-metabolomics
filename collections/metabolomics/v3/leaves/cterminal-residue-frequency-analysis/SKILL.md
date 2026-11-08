---
name: cterminal-residue-frequency-analysis
description: 'Use when comparing de novo peptide sequencing models for tryptic bias:
  if predicted peptides from a fine-tuned vs. pretrained transformer model (e.g.,
  Casanovo vs. Casanovone) should be checked for reduced C-terminal K/R enrichment
  on non-enzymatic spectra.'
when_to_use_negative:
- The predicted peptides come from fully tryptic digests with enforced cleavage rules,
  where C-terminal K/R is imposed by design and carries no signal about model bias
- The goal is scoring sequence accuracy itself (use peptide- and amino-acid-level
  precision from 'casanovo sequence --evaluate' instead of residue-position statistics)
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Casanovo
  role: Transformer de novo sequencer whose predictions (pretrained and non-enzymatic
    fine-tuned checkpoints, decoded with beam search) provide the peptide strings
    for C-terminal K/R frequency tabulation
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:3e2c185eab86e4efdf93b272f145fcb20494b4ac6f0dfc88e69089eb2539c32f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# C-terminal residue frequency analysis

## Summary

Summarizes the frequency of C-terminal lysine (K) and arginine (R) in predicted peptide sequences to quantify enzymatic (tryptic) bias in de novo sequencing models, as used to evaluate the non-enzymatic Casanovone fine-tuning of Casanovo.

## When to use

Use when comparing de novo peptide sequencing models for tryptic bias: if predicted peptides from a fine-tuned vs. pretrained transformer model (e.g., Casanovo vs. Casanovone) should be checked for reduced C-terminal K/R enrichment on non-enzymatic spectra.

## When NOT to use

- The predicted peptides come from fully tryptic digests with enforced cleavage rules, where C-terminal K/R is imposed by design and carries no signal about model bias
- The goal is scoring sequence accuracy itself (use peptide- and amino-acid-level precision from 'casanovo sequence --evaluate' instead of residue-position statistics)

## Inputs

- Predicted peptide sequences (strings) from a de novo sequencing model, e.g. Casanovo output on an MGF test split
- Annotated MGF non-enzymatic test split (100,000 PSMs, MassIVE MSV000094014 / doi:10.25345/C5KS6JG0W)
- Model checkpoints: pretrained Casanovo and fine-tuned non-enzymatic (Casanovone) .ckpt files

## Outputs

- C-terminal lysine/arginine frequency distribution of predicted peptides per model (Fig. 3a-b style summary)
- Comparative tabulation showing reduced C-terminal K/R frequencies for the fine-tuned model versus the tryptic-biased original model

## How to apply

Run both the pretrained Casanovo checkpoint and the fine-tuned non-enzymatic checkpoint over the same annotated MGF test split (e.g., the 100,000-PSM non-enzymatic test set from MassIVE MSV000094014) using beam search decoding ('casanovo sequence --evaluate'). From each model's predicted peptide strings, tabulate the C-terminal residue distribution, focusing on lysine and arginine frequencies, since standard models trained on tryptic MassIVE-KB PSMs over-predict K/R at the C-terminus. Compare the two distributions side-by-side to reproduce the reduced C-terminal K/R frequencies reported in Fig. 3a-b; a successful fine-tuning shows visibly reduced C-terminal K/R in the non-enzymatic model alongside improved peptide-level average precision (0.83 vs 0.60).

## Related tools

- **Casanovo** (Transformer de novo sequencer whose predictions (pretrained and non-enzymatic fine-tuned checkpoints, decoded with beam search) provide the peptide strings for C-terminal K/R frequency tabulation) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo sequence --evaluate --model casanovone.best.ckpt nonenzymatic_test.mgf
```

## Evaluation signals

- Fine-tuned model reaches peptide-level average precision of approximately 0.83 versus approximately 0.60 for the original Casanovo model on the 100,000-PSM non-enzymatic test split
- C-terminal lysine/arginine frequencies in the fine-tuned model's predictions are visibly reduced relative to the pretrained model, matching Fig. 3a-b
- Both models are evaluated on the identical test split with beam search decoding ('casanovo sequence --evaluate'), so distribution differences are attributable to the model, not the data
- The checkpoint kept is the one with the lowest validation loss among snapshots taken every val_check_interval during fine-tuning

## Limitations

- Frequency analysis alone does not certify sequencing correctness; it must be paired with peptide-level and amino-acid-level precision measures emitted during 'casanovo sequence --evaluate' evaluation
- Requires a held-out non-enzymatic test set with annotated peptides; cannot be interpreted on spectra lacking ground-truth labels
- The field lacks a rigorous method for confidence estimation for de novo sequencing, which is a key bottleneck when interpreting model predictions

## Evidence

- [methods] Summarize the C-terminal lysine/arginine frequency distribution of predicted peptides for both models to reproduce the reduced K/R frequencies reported in Fig. 3a-b.: [evidence span withheld — anchor status: not_found]
- [results] Fine-tuning improves peptide-level precision on non-tryptic spectra: [evidence span withheld — anchor status: not_found]
- [readme] Casanovo applies to non-enzymatic settings: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset, and is particularly useful for immunopeptidomics"
- [methods] Evaluation command: [evidence span withheld — anchor status: not_found]
