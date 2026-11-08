---
name: peptide-precision-benchmark-comparison
description: Use when you must benchmark de novo sequencing tools on annotated MS/MS
  spectra across multiple species, holding out whole species to test cross-species
  generalization with peptide-level precision.
when_to_use_negative:
- The goal is database searching of spectra against a known protein FASTA (use Casanovo-DB
  mode or a standard search engine instead of a de novo benchmark)
- Data are DIA rather than DDA spectra — DeepNovo's DDA repository does not handle
  DIA (see nh2tran/DeepNovo-DIA)
- Only a single species is of interest with no held-out organism, so leave-one-species-out
  cross-validation cannot measure cross-species generalization
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer-based de novo sequencer trained and evaluated on each cross-validation
    fold
  repo: https://github.com/Noble-Lab/casanovo
- name: PointNovo
  role: Baseline de novo sequencer run on the same held-out test spectra for comparison
- name: DeepNovo
  role: Baseline CNN+RNN de novo sequencer run on the same held-out test spectra
  repo: nh2tran/DeepNovo
- name: Novor
  role: Decision-tree baseline de novo sequencer run on the same held-out test spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:d4965df1ea8135fbc7c6484d1974df3c6796d6fd20bad3b946195d5af8f886f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - DeepNovo
  - Novor
  - PointNovo
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# peptide-precision-benchmark-comparison

## Summary

A cross-species de novo peptide sequencing benchmark protocol that compares transformer-based Casanovo against PointNovo, DeepNovo, and Novor on the nine-species Tran et al. dataset, scoring each tool by peptide-level average precision under leave-one-species-out cross-validation.

## When to use

Use when you must benchmark de novo sequencing tools on annotated MS/MS spectra across multiple species, holding out whole species to test cross-species generalization with peptide-level precision.

## When NOT to use

- The goal is database searching of spectra against a known protein FASTA (use Casanovo-DB mode or a standard search engine instead of a de novo benchmark)
- Data are DIA rather than DDA spectra — DeepNovo's DDA repository does not handle DIA (see nh2tran/DeepNovo-DIA)
- Only a single species is of interest with no held-out organism, so leave-one-species-out cross-validation cannot measure cross-species generalization

## Inputs

- Annotated MGF files for the nine-species benchmark (MassIVE MSV000090982)
- Casanovo model trained on 30 million confident PSMs from the MassIVE-KB spectral library
- Pre-trained or locally installed PointNovo, DeepNovo, and Novor

## Outputs

- Per-tool, per-species peptide-level average precision values
- denovo_benchmark_average_precision.tsv with aggregated average precisions across nine folds

## How to apply

Download the annotated MGF files for the nine-species benchmark from MassIVE dataset MSV000090982, which the Casanovo paper provides for cross-tool evaluation. Set up cross-validation splits that hold out each of the nine species in turn as the test set while training on the remaining eight species, so that generalization to unseen organisms is measured rather than memorization. Train/evaluate Casanovo (v2.0-style transformer setup, trained on MassIVE-KB spectra) on each fold and predict peptides for the held-out species, then run PointNovo, DeepNovo, and Novor on the same held-out test spectra for every fold to keep the comparison fair. Compute peptide-level average precision per tool per species, aggregate across the nine folds, and write results to denovo_benchmark_average_precision.tsv.

## Related tools

- **Casanovo** (Transformer-based de novo sequencer trained and evaluated on each cross-validation fold) — https://github.com/Noble-Lab/casanovo
- **PointNovo** (Baseline de novo sequencer run on the same held-out test spectra for comparison)
- **DeepNovo** (Baseline CNN+RNN de novo sequencer run on the same held-out test spectra) — nh2tran/DeepNovo
- **Novor** (Decision-tree baseline de novo sequencer run on the same held-out test spectra)

## Examples

```
casanovo run --config casanovo.yaml --mode de_novo --output denovo_predictions/ *.mgf
```

## Evaluation signals

- Reproduced peptide-level average precisions should match reported Fig. 2a values within reasonable tolerance: Casanovo 0.81, PointNovo 0.74, DeepNovo 0.70, Novor 0.58
- Casanovo's average precision should exceed all three baselines on the nine-species benchmark
- Each tool must be evaluated on exactly the same held-out test spectra per fold
- All nine species appear exactly once as a held-out test set across the cross-validation folds

## Limitations

- The paper notes the field lacks 'an exhaustive and rigorous benchmark comparison of this growing field of tools', so this nine-species comparison is not exhaustive
- No rigorous method for confidence estimation for de novo sequencing exists, complicating thresholded comparisons
- Whether Casanovo outperforms standard database search in statistical power for a given DDA dataset remains an open question

## Evidence

- [abstract] cross-species benchmark performance: "demonstrate that the model outperforms several state-of-the-art methods on a cross-species benchmark dataset"
- [methods] benchmark dataset availability: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [methods] training data provenance: "improved performance of Casanovo by training on the MassIVE-KB data"
- [intro] baseline descriptions: "PointNovo17, the successor to DeepNovo, leverages an order-invariant network architecture to model high-resolution MS/MS spectra18."
- [discussion] benchmark gap: "Clearly, the field would benefit from an exhaustive and rigorous benchmark comparison of this growing field of tools."
- [readme] DeepNovo tool scope: [evidence span withheld — anchor status: external_doc]
