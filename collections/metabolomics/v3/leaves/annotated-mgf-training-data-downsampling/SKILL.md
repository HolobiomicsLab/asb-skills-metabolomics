---
name: annotated-mgf-training-data-downsampling
description: 'Use when comparing two de novo sequencing training sets of different
  sizes: you need size-matched training data to test whether data quality (e.g., curated
  MassIVE-KB PSMs) rather than peptide count drives performance.'
when_to_use_negative:
- You only need to run de novo peptide sequencing on new spectra — use Casanovo in
  de novo mode instead of re-training on downsampled data
- The research question concerns the effect of dataset size itself, in which case
  the full 30-million-PSM MassIVE-KB training set should be used without downsampling
- Input spectra are unannotated or not in annotated MGF format, since training requires
  labeled PSMs rather than raw MS/MS files
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3473
tools:
- name: Casanovo
  role: Transformer model trained on the downsampled annotated MGF data and evaluated
    with peptide precision / average precision on the nine-species benchmark
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:dd2e5b571ba4cc5b3140dc090d78bed0ebb2681aa6dc73d9f79adba50751c21d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Annotated MGF training data downsampling

## Summary

Downsamples the MassIVE-KB v1 30-million-PSM annotated MGF training set to a peptide count matched to a smaller baseline training set (~239,697 vs ~246,713 peptides) so that a Casanovo-vs-Casanovobm comparison isolates training-data quality from dataset size.

## When to use

Use when comparing two de novo sequencing training sets of different sizes: you need size-matched training data to test whether data quality (e.g., curated MassIVE-KB PSMs) rather than peptide count drives performance.

## When NOT to use

- You only need to run de novo peptide sequencing on new spectra — use Casanovo in de novo mode instead of re-training on downsampled data
- The research question concerns the effect of dataset size itself, in which case the full 30-million-PSM MassIVE-KB training set should be used without downsampling
- Input spectra are unannotated or not in annotated MGF format, since training requires labeled PSMs rather than raw MS/MS files

## Inputs

- MassIVE-KB v1 30-million-PSM annotated MGF training data (Zenodo 10.5281/zenodo.14973856)
- MassIVE-KB v1 training/validation/test splits (Zenodo 10.5281/zenodo.14967861)
- Baseline training set with known peptide count for size matching (~246,713 peptides, e.g., Casanovobm)
- Revised nine-species benchmark annotated MGF files from MassIVE MSV000090982

## Outputs

- Downsampled annotated MGF training set of ~239,697 peptides
- Trained Casanovo checkpoint with lowest validation loss (<output_root>.best.ckpt)
- average_precision_comparison.csv comparing downsampled MassIVE-KB vs size-matched baseline average precision

## How to apply

Download the MassIVE-KB v1 30-million-PSM annotated MGF data and its train/validation/test splits from Zenodo (10.5281/zenodo.14973856 and 10.5281/zenodo.14967861). Downsample the training set to approximately 239,697 peptides so the peptide count matches the baseline training set's ~246,713 peptides, keeping the original split files intact. Train Casanovo on the downsampled annotated MGF data with 'casanovo train', retaining the checkpoint with the lowest validation loss saved as <output_root>.best.ckpt. Evaluate both the downsampled MassIVE-KB model and the size-matched baseline on the revised nine-species benchmark (annotated MGF files from MassIVE MSV000090982) using the cross-validated protocol of training on eight species and testing on the held-out species, computing peptide precision and average precision. Tabulate the average precision of both models and save the comparison as average_precision_comparison.csv; the rationale is that the paper identifies the carefully curated, stringently FDR-controlled MassIVE-KB collection as particularly good training data.

## Related tools

- **Casanovo** (Transformer model trained on the downsampled annotated MGF data and evaluated with peptide precision / average precision on the nine-species benchmark) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo train -c casanovo.yaml -t massivekb_downsampled_239697.mgf -v massivekb_val.mgf -o out/
```

## Evaluation signals

- Downsampled training set peptide count is ~239,697, matching the baseline's ~246,713 peptides within a small tolerance
- Retained checkpoint is the one with the lowest validation loss and is saved as <output_root>.best.ckpt
- Downsampled MassIVE-KB model reaches average precision ~0.90 vs ~0.83 for the size-matched Casanovobm baseline on the nine-species benchmark
- Evaluation follows the cross-validated protocol: train on eight species, test on the held-out species, for each of the nine species
- Comparison results are written to average_precision_comparison.csv with peptide precision and average precision per model

## Limitations

- The mass spectrometry field lacks an exhaustive and rigorous benchmark comparison of de novo sequencing tools, so cross-benchmark conclusions should be drawn cautiously
- No rigorous method for confidence estimation for de novo sequencing exists, complicating absolute performance assessment
- The MassIVE-KB advantage is attributed to curation from a massive collection of 669 million spectra with stringent FDR control; downsampling cannot reproduce the underlying spectral diversity, only the peptide count

## Evidence

- [discussion] MassIVE-KB quality claim: "Our experiments suggest that the carefully curated MassIVE-KB collection provides particularly good training data."
- [intro] MassIVE-KB training scale: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23"
- [abstract] Casanovo training scale: "We train a Casanovo model from 30 million labeled spectra"
- [methods] Benchmark availability: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [readme] Casanovo capability: "Powered by a transformer neural network, Casanovo "translates" peaks in MS/MS spectra into amino acid sequences"
