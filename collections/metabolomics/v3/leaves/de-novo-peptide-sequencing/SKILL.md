---
name: de-novo-peptide-sequencing
description: Use when DDA bottom-up MS/MS spectra (MGF annotated files, immunopeptidomics,
  metaproteomics, or 'dark proteome' unidentified clusters) must be assigned peptide
  sequences without a known protein database or to detect variants, PTMs, and foreign
  peptides.
when_to_use_negative:
- The set of proteins in the sample is known in advance and only standard identification
  is needed — database searching (SEQUEST/Tide-style) is the appropriate choice, and
  an open question remains whether de novo outperforms standard database search in
  statistical power for a given DDA dataset.
- Input spectra contain too few peaks or are low quality — the min_peaks option discards
  such spectra, so results on them would be unreliable.
- Data are DIA acquisitions — this skill targets DDA bottom-up data; DeepNovo's DIA
  framework is maintained in a separate repository.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Primary transformer-based de novo sequencer; supports beam search decoding,
    fine-tuning for non-enzymatic peptides, mzTab export, and database-search mode
    (casanovo db_search)
  repo: https://github.com/Noble-Lab/casanovo
- name: DeepNovo
  role: CNN+RNN baseline de novo sequencer for cross-tool benchmark comparison on
    held-out species test splits
  repo: https://github.com/nh2tran/DeepNovo
- name: Novor
  role: Decision-tree-based de novo baseline tool run on the same held-out benchmark
    spectra
- name: PointNovo
  role: Order-invariant network baseline for high-resolution MS/MS de novo sequencing,
    used as comparison on the nine-species benchmark
- name: limelight-import-casanovo
  role: Converts Casanovo mzTab results to Limelight XML for upload to the Limelight
    web application
  repo: https://github.com/yeastrc/limelight-import-casanovo
merged_aliases:
- de-novo-peptide-sequencing-benchmarking
merged_alias_records:
- alias: de-novo-peptide-sequencing-model-training
  slug: de-novo-peptide-sequencing-model-training
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
- alias: de-novo-peptide-sequencing-spectra-annotation
  slug: de-novo-peptide-sequencing-spectra-annotation
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:32ba20b72a185bdbf04e6ba290727fe5a6c8a658796d547a2d2b73302e8dfabb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - DeNovoGUI
  - DeepNovo
  - MS-GF+
  - Novor
  - PeptideShaker
  - PointNovo
  - ProteoWizard
  - SMSNet
  - SearchGUI
  - X!Tandem
  - limelight-import-casanovo
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# de-novo-peptide-sequencing

## Summary

Assign amino acid sequences directly to tandem MS/MS spectra without a protein database by treating the task as machine translation with a transformer model (Casanovo), which was trained on 30 million labeled PSMs from MassIVE-KB and outperforms Novor, DeepNovo, and PointNovo on a nine-species cross-species benchmark.

## When to use

Use when DDA bottom-up MS/MS spectra (MGF annotated files, immunopeptidomics, metaproteomics, or 'dark proteome' unidentified clusters) must be assigned peptide sequences without a known protein database or to detect variants, PTMs, and foreign peptides.

## When NOT to use

- The set of proteins in the sample is known in advance and only standard identification is needed — database searching (SEQUEST/Tide-style) is the appropriate choice, and an open question remains whether de novo outperforms standard database search in statistical power for a given DDA dataset.
- Input spectra contain too few peaks or are low quality — the min_peaks option discards such spectra, so results on them would be unreliable.
- Data are DIA acquisitions — this skill targets DDA bottom-up data; DeepNovo's DIA framework is maintained in a separate repository.

## Inputs

- annotated MGF files of MS/MS spectra (e.g., nine-species benchmark from MassIVE MSV000090982)
- DDA bottom-up tandem mass spectra (mzML/mgf converted via msconvert)
- pretrained Casanovo model weights file
- MassIVE-KB v1 30-million-PSM training data (Zenodo 10.5281/zenodo.14973856) with train/validation/test splits (Zenodo 10.5281/zenodo.14967861)
- protein FASTA database for post-hoc peptide mapping

## Outputs

- per-spectrum peptide predictions with confidence scores in mzTab format
- peptide-level average precision / precision tables (e.g., denovo_benchmark_average_precision.tsv)
- cluster-level consensus peptide assignments (e.g., gleams_cluster_assignments_summary.tsv)
- Limelight XML files for web-based result upload

## How to apply

Preprocess spectra (spectrum_utils; apply min_peaks/max_peaks options to discard low-quality spectra; note that the precursor mass filter is not applied in de novo mode), then run a transformer-based de novo sequencer such as Casanovo with beam search decoding rather than greedy decoding to predict the best peptide per spectrum. For benchmarking, use cross-validated splits of the nine-species annotated MGF files from MassIVE MSV000090982, holding out each species in turn and computing peptide-level average precision (Casanovobm 0.81 vs PointNovo 0.74, DeepNovo 0.70, Novor 0.58). For specialized tasks such as immunopeptidomics, fine-tune a pre-trained non-enzymatic model instead of training from scratch. Export predictions as mzTab files for downstream mapping to proteome FASTA, consensus clustering (e.g., GLEAMS cluster plurality vote), or Limelight XML conversion.

## Related tools

- **Casanovo** (Primary transformer-based de novo sequencer; supports beam search decoding, fine-tuning for non-enzymatic peptides, mzTab export, and database-search mode (casanovo db_search)) — https://github.com/Noble-Lab/casanovo
- **DeepNovo** (CNN+RNN baseline de novo sequencer for cross-tool benchmark comparison on held-out species test splits) — https://github.com/nh2tran/DeepNovo
- **Novor** (Decision-tree-based de novo baseline tool run on the same held-out benchmark spectra)
- **PointNovo** (Order-invariant network baseline for high-resolution MS/MS de novo sequencing, used as comparison on the nine-species benchmark)
- **limelight-import-casanovo** (Converts Casanovo mzTab results to Limelight XML for upload to the Limelight web application) — https://github.com/yeastrc/limelight-import-casanovo

## Examples

```
casanovo train  # train on annotated MGF training data, retaining the lowest-validation-loss checkpoint <output_root>.best.ckpt
```

## Evaluation signals

- Reproduced peptide-level average precision on the nine-species cross-validation should match the reported Fig. 2a values within tolerance: Casanovobm 0.81, PointNovo 0.74, DeepNovo 0.70, Novor 0.58.
- On the revised benchmark, a size-matched downsampled MassIVE-KB-trained model should reach ~0.90 average precision vs ~0.83 for Casanovobm, confirming data quality (not size) drives gains.
- Output mzTab files must parse and per-spectrum predictions deduplicate to expected unique peptide counts (e.g., 26,377 unique peptides and 2459 human-proteome matches for MDA-MB-231 immunopeptidomics).
- Training runs must retain the checkpoint with the lowest validation loss saved as <output_root>.best.ckpt.
- Dark-proteome runs should reproduce cluster-consensus counts (89,250 assigned clusters, 16%; 52,523 clusters after proteome-first filtering) and 196,724 assigned spectra, with ~83.4% of predicted substitutions consistent with single-nucleotide substitutions.

## Limitations

- No rigorous method for confidence estimation for de novo sequencing exists — described as one of the key bottlenecks in the field.
- The precursor mass filter is not applied in de novo mode, and a rigorous exhaustive benchmark comparison of the growing set of de novo tools is still lacking.
- Target-decoy matching of predictions misses the power of de novo sequencing to assign peptides to foreign spectra; the impact of model parameter count on performance is an open question.

## Evidence

- [abstract] transformer architecture for peak-to-peptide translation: "uses a transformer neural network architecture to translate the sequence of peaks in a tandem mass spectrum into the sequence of amino acids"
- [methods] fine-tuning instead of retraining: "you can fine-tune an existing pre-trained Casanovo model on data with new PTMs instead of training from scratch"
- [methods] de novo mode precursor mass filter behavior: "The precursor mass filter is no longer applied in *de novo* mode"
- [methods] mzTab export step: "Export spectrum predictions to mzTab files."
- [readme] scope of applicable data: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset"
