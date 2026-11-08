---
name: average-precision-comparison-analysis
description: Use when two de novo sequencing models differ in training-set provenance
  (e.g. curated MassIVE-KB vs custom-labeled data) and you must test whether performance
  gains stem from data quality rather than peptide-count scale.
when_to_use_negative:
- The models being compared were trained on training sets of different sizes — peptide-count
  scale then confounds the data-quality question, so downsample or expand one set
  to match first
- The benchmark spectra are not provided as annotated MGF files with known peptide
  labels — peptide precision and average precision cannot be computed without ground-truth
  peptides
- The goal is running a de novo search on new spectra rather than benchmarking model
  training-data composition (use Casanovo's de novo or db_search mode instead)
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer de novo sequencing model trained on the downsampled MassIVE-KB
    annotated MGF data and evaluated with peptide precision and average precision
    on the nine-species benchmark
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:b300891cb49a423d5a08575400c96a9992f0526e669e2da99d749554fecc16a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# average-precision-comparison-analysis

## Summary

Protocol for disentangling training-data quality from dataset size in de novo peptide sequencing by comparing peptide precision and average precision of models trained on size-matched training sets (e.g. downsampled MassIVE-KB vs Casanovobm) on the nine-species cross-validated benchmark.

## When to use

Use when two de novo sequencing models differ in training-set provenance (e.g. curated MassIVE-KB vs custom-labeled data) and you must test whether performance gains stem from data quality rather than peptide-count scale.

## When NOT to use

- The models being compared were trained on training sets of different sizes — peptide-count scale then confounds the data-quality question, so downsample or expand one set to match first
- The benchmark spectra are not provided as annotated MGF files with known peptide labels — peptide precision and average precision cannot be computed without ground-truth peptides
- The goal is running a de novo search on new spectra rather than benchmarking model training-data composition (use Casanovo's de novo or db_search mode instead)

## Inputs

- MassIVE-KB v1 30-million-PSM annotated MGF training data (Zenodo 10.5281/zenodo.14973856)
- MassIVE-KB v1 training/validation/test split files (Zenodo 10.5281/zenodo.14967861)
- Revised nine-species benchmark annotated MGF files from MassIVE dataset MSV000090982
- Pre-trained Casanovobm baseline model trained on ~246,713 peptides

## Outputs

- Casanovo checkpoint with lowest validation loss (<output_root>.best.ckpt)
- Per-species peptide precision and average precision metrics for both models
- average_precision_comparison.csv tabulating average precision of the downsampled MassIVE-KB model vs the size-matched Casanovobm model

## How to apply

Download the MassIVE-KB v1 30-million-PSM annotated MGF training data and its train/validation/test splits from Zenodo (10.5281/zenodo.14973856 and 10.5281/zenodo.14967861). Downsample the training set to approximately 239,697 peptides so the peptide count matches the Casanovobm baseline's ~246,713 peptides, isolating quality as the only variable. Train a Casanovo model on the downsampled annotated MGF data with 'casanovo train', retaining the checkpoint with the lowest validation loss saved as <output_root>.best.ckpt. Evaluate both the downsampled MassIVE-KB model and the size-matched Casanovobm baseline on the revised nine-species benchmark annotated MGF files (MassIVE MSV000090982) using the cross-validated protocol of training on eight species and testing on the held-out species. Compute peptide precision and average precision for both models, then tabulate the comparison as average_precision_comparison.csv.

## Related tools

- **Casanovo** (Transformer de novo sequencing model trained on the downsampled MassIVE-KB annotated MGF data and evaluated with peptide precision and average precision on the nine-species benchmark) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo train --config casanovo_config.yaml massive_kb_downsampled.mgf
```

## Evaluation signals

- Peptide counts of the two training sets are matched within ~7,000 peptides (downsampled MassIVE-KB ~239,697 vs Casanovobm ~246,713)
- The retained Casanovo checkpoint corresponds to the lowest validation loss and is saved as <output_root>.best.ckpt
- The downsampled MassIVE-KB model reaches average precision ~0.90 on the nine-species benchmark
- The size-matched Casanovobm baseline stays at average precision ~0.83, and the gap (0.83 → 0.90) is reproduced under the eight-species-train / one-species-test cross-validated protocol
- average_precision_comparison.csv contains average precision rows for both models across the held-out species

## Limitations

- The comparison relies on a single size-matching target (~239,697 peptides); other downsample ratios or downsampling strategies might shift the average precision gap
- The article flags the field-wide lack of a rigorous confidence-estimation method for de novo sequencing, so average precision comparisons do not address per-spectrum score calibration
- Whether Casanovo outperforms standard database search in statistical power for a given DDA dataset remains an open question, so this benchmark does not by itself justify replacing database searching

## Evidence

- [discussion] Our experiments suggest that the carefully curated MassIVE-KB collection provides particularly good training data.: "Our experiments suggest that the carefully curated MassIVE-KB collection provides particularly good training data."
- [methods] improved performance of Casanovo by training on the MassIVE-KB data: "improved performance of Casanovo by training on the MassIVE-KB data"
- [methods] available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [readme] Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing.: "Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing."
