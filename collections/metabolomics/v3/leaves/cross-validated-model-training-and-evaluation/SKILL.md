---
name: cross-validated-model-training-and-evaluation
description: Use when benchmarking a de novo sequencing model's generalization across
  species — i.e., annotated MS/MS spectra (MGF) must be split so each organism is
  held out in turn as a test fold while training on the remaining species.
when_to_use_negative:
- The dataset is DIA (data-independent acquisition) — this skill targets DDA bottom-up
  tandem MS data, and DeepNovo's authors maintain a separate repository for DIA analysis
- The protein set in the sample is fully known and a conventional database search
  (e.g., SEQUEST-style) would suffice — the benchmarking here is for de novo sequencing
  of spectra not covered by a known database
- You need per-spectrum confidence scores calibrated to a known FDR — the article
  notes the field lacks a rigorous method for confidence estimation for de novo sequencing,
  so this benchmark's average-precision metric is not a substitute for that
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer-based de novo sequencer trained per cross-validation fold on the
    eight-species training split and used to predict peptides for the held-out species
  repo: https://github.com/Noble-Lab/casanovo
- name: DeepNovo
  role: Baseline de novo sequencing tool run on the same held-out test spectra in
    every fold
  repo: https://github.com/nh2tran/DeepNovo
- name: PointNovo
  role: Baseline de novo sequencing tool (successor to DeepNovo) run on the same held-out
    test spectra in every fold
- name: Novor
  role: Baseline de novo sequencing tool using a decision-tree score function with
    dynamic programming, run on the same held-out test spectra in every fold
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:2f7694c8226c72d558d3d0e45c3fdefec383ea07e82cbad2c2f82865a5554722
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - DeepNovo
  - Novor
  - PointNovo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-validated-model-training-and-evaluation

## Summary

Train and evaluate a de novo peptide sequencing model (Casanovo, transformer-based) under species-level cross-validation against competing tools (PointNovo, DeepNovo, Novor), scoring peptide-level average precision on held-out spectra from annotated MGF benchmark data.

## When to use

Use when benchmarking a de novo sequencing model's generalization across species — i.e., annotated MS/MS spectra (MGF) must be split so each organism is held out in turn as a test fold while training on the remaining species.

## When NOT to use

- The dataset is DIA (data-independent acquisition) — this skill targets DDA bottom-up tandem MS data, and DeepNovo's authors maintain a separate repository for DIA analysis
- The protein set in the sample is fully known and a conventional database search (e.g., SEQUEST-style) would suffice — the benchmarking here is for de novo sequencing of spectra not covered by a known database
- You need per-spectrum confidence scores calibrated to a known FDR — the article notes the field lacks a rigorous method for confidence estimation for de novo sequencing, so this benchmark's average-precision metric is not a substitute for that

## Inputs

- Annotated MGF files for the nine-species benchmark (MassIVE dataset MSV000090982)
- MassIVE-KB v1 30 million PSM training/validation/test splits (Zenodo 10.5281/zenodo.14967861) for training the Casanovo model
- Cross-validation fold definitions (one held-out species per fold, nine folds total)

## Outputs

- Trained Casanovo model checkpoints per cross-validation fold
- Peptide predictions from Casanovo, PointNovo, DeepNovo, and Novor for each held-out species' test spectra
- Aggregated per-tool and per-species peptide-level average precision table (denovo_benchmark_average_precision.tsv)

## How to apply

Download the annotated MGF files for the nine-species benchmark from MassIVE dataset MSV000090982, then set up leave-one-species-out cross-validation: hold out each of the nine species as the test set while training on the remaining eight. Train Casanovo (transformer-based de novo sequencer, v2.0-style setup, trained on 30 million confident PSMs from MassIVE-KB with beam search decoding) on each fold and predict peptides for the held-out species' spectra. Run PointNovo, DeepNovo, and Novor on the identical held-out test spectra for every fold so all tools see the same inputs. Compute peptide-level average precision for each tool on each species test split, then aggregate across the nine folds. The rationale for the leave-one-species-out design is that it measures cross-species generalization — the model is never tested on a species it was trained on, unlike a random spectral split. Configurable spectrum filters (min_peaks, max_peaks, minimum peptide length) should be set consistently across all tools to keep the comparison fair.

## Related tools

- **Casanovo** (Transformer-based de novo sequencer trained per cross-validation fold on the eight-species training split and used to predict peptides for the held-out species) — https://github.com/Noble-Lab/casanovo
- **DeepNovo** (Baseline de novo sequencing tool run on the same held-out test spectra in every fold) — https://github.com/nh2tran/DeepNovo
- **PointNovo** (Baseline de novo sequencing tool (successor to DeepNovo) run on the same held-out test spectra in every fold)
- **Novor** (Baseline de novo sequencing tool using a decision-tree score function with dynamic programming, run on the same held-out test spectra in every fold)

## Evaluation signals

- Per-fold held-out species' spectra are disjoint from training species — verify no spectral leakage across the nine leave-one-species-out splits
- Aggregated peptide-level average precisions should match the reported values (Casanovo 0.81, PointNovo 0.74, DeepNovo 0.70, Novor 0.58 from Fig. 2a) within reasonable tolerance
- All four tools must be run on the identical held-out test spectra for each fold before computing average precision
- Output table denovo_benchmark_average_precision.tsv contains 9 species x 4 tool entries, plus the aggregated average for each tool
- Casanovo predictions are generated with the same beam-search decoding and MassIVE-KB-trained model configuration across all folds

## Limitations

- The field lacks an exhaustive and rigorous benchmark comparison of the growing set of de novo sequencing tools, so tolerance for reproduction must be generous rather than exact
- There is no rigorous method for confidence estimation for de novo sequencing — average precision is a ranking metric, not a calibrated per-spectrum confidence
- Performance depends strongly on training data quality; the MassIVE-KB collection (669 million spectra with stringent FDR control) is cited as particularly good training data, so results may not transfer to models trained on smaller/noisier sets
- Casanovo is designed for data-dependent acquisition, bottom-up tandem MS datasets; it is not validated for DIA inputs in this skill

## Evidence

- [methods] On the nine-species benchmark the reproduced peptide-level average precisions should match the reported values (Casanovo 0.81, PointNovo 0.74, DeepNovo 0.70, Novor 0.58 from Fig. 2a): "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [intro] Training data expansion is central to the model being benchmarked.: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure"
- [abstract] The benchmark's headline claim is cross-species outperformance of prior tools.: "demonstrate that the model outperforms several state-of-the-art methods on a cross-species benchmark dataset"
- [intro] PointNovo is described as an order-invariant network for high-resolution spectra, defining its baseline role in the comparison.: "PointNovo17, the successor to DeepNovo, leverages an order-invariant network architecture to model high-resolution MS/MS spectra"
- [methods] Fine-tuning rather than retraining from scratch is the recommended path when adapting the model per fold.: "you can fine-tune an existing pre-trained Casanovo model on data with new PTMs instead of training from scratch"
- [readme] The Casanovo README defines the tool being trained and evaluated.: "Powered by a transformer neural network, Casanovo "translates" peaks in MS/MS spectra into amino acid sequences with remarkable precision."
- [discussion] The article explicitly calls for rigorous tool benchmarking, motivating the cross-validated evaluation design.: "Clearly, the field would benefit from an exhaustive and rigorous benchmark comparison of this growing field of tools."
