---
name: cross-species-benchmark-peptide-precision-evaluation
description: Use when comparing two de novo sequencing models (e.g., two Casanovo
  variants trained on different training sets) with a controlled cross-species evaluation,
  or when reporting peptide precision and average precision on held-out species from
  MSV000090982.
when_to_use_negative:
- You need protein-level or FDR-estimated identifications rather than de novo precision
  metrics — database searching (e.g., 'casanovo db_search') targets a different question.
- The comparison question is about model architecture rather than training-set size
  or quality — cross-species precision evaluation here is designed for matched-training-set
  comparisons.
- Your test data are DIA spectra or non-enzymatic (immunopeptidomics) spectra requiring
  the fine-tuned non-enzymatic Casanovo variant instead of the standard benchmark
  protocol.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: Casanovo
  role: Transformer-based de novo sequencing model trained on downsampled MassIVE-KB
    annotated MGF data and evaluated on the nine-species benchmark
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:3b212e1868f953333e40d280406471358d2264a79cfd46c465cee067e5d0e046
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-species-benchmark-peptide-precision-evaluation

## Summary

Evaluate a de novo peptide sequencing model by cross-validated testing on a multi-species benchmark of annotated MGF files, computing peptide precision and average precision for each held-out species. This protocol enables size- and quality-matched model comparisons such as downsampled MassIVE-KB-trained Casanovo versus a size-matched baseline.

## When to use

Use when comparing two de novo sequencing models (e.g., two Casanovo variants trained on different training sets) with a controlled cross-species evaluation, or when reporting peptide precision and average precision on held-out species from MSV000090982.

## When NOT to use

- You need protein-level or FDR-estimated identifications rather than de novo precision metrics — database searching (e.g., 'casanovo db_search') targets a different question.
- The comparison question is about model architecture rather than training-set size or quality — cross-species precision evaluation here is designed for matched-training-set comparisons.
- Your test data are DIA spectra or non-enzymatic (immunopeptidomics) spectra requiring the fine-tuned non-enzymatic Casanovo variant instead of the standard benchmark protocol.

## Inputs

- Revised nine-species benchmark annotated MGF files (MassIVE MSV000090982)
- MassIVE-KB v1 30-million-PSM annotated MGF training data (Zenodo 10.5281/zenodo.14973856)
- MassIVE-KB training/validation/test splits (Zenodo 10.5281/zenodo.14967861)
- Size-matched baseline model checkpoint (e.g., Casanovobm)
- Downsampled training set (~239,697 peptides) to match baseline peptide count

## Outputs

- Per-species peptide precision values for each model
- Per-species and overall average precision values for each model
- Comparison table saved as average_precision_comparison.csv
- Trained model checkpoint with lowest validation loss (<output_root>.best.ckpt)

## How to apply

Obtain the revised nine-species benchmark as annotated MGF files from MassIVE dataset MSV000090982. Apply the cross-validated protocol: train the model on data from eight species and test on the held-out species, repeating so each species is held out once. For the data-quality question, downsample the MassIVE-KB v1 30-million-PSM annotated MGF training set (from Zenodo 10.5281/zenodo.14973856 with splits from 10.5281/zenodo.14967861) to match the baseline's peptide count (~239,697 vs ~246,713 peptides), train with 'casanovo train' keeping the checkpoint with lowest validation loss, then evaluate both models identically. Tabulate peptide precision and average precision per species and save the aggregate comparison (e.g., average_precision_comparison.csv); a correct application reproduces ~0.90 average precision for the downsampled MassIVE-KB model versus ~0.83 for the size-matched baseline.

## Related tools

- **Casanovo** (Transformer-based de novo sequencing model trained on downsampled MassIVE-KB annotated MGF data and evaluated on the nine-species benchmark) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo train config.yml   # trains on the downsampled MassIVE-KB annotated MGF data, retaining the lowest-validation-loss checkpoint <output_root>.best.ckpt
```

## Evaluation signals

- Downsampled MassIVE-KB-trained Casanovo reaches average precision ≈ 0.90 on the nine-species benchmark versus ≈ 0.83 for the size-matched Casanovobm model.
- Training-set peptide counts are matched before comparison (~239,697 for downsampled MassIVE-KB vs ~246,713 for Casanovobm).
- Every species in the benchmark is used exactly once as the held-out test set in the cross-validated (train-on-eight-species) protocol.
- The retained model checkpoint is the one with the lowest validation loss, saved as <output_root>.best.ckpt.
- The comparison output average_precision_comparison.csv contains both models' average precision values computed with the identical evaluation protocol.

## Limitations

- Results depend on the specific curated MassIVE-KB training collection; the paper attributes the advantage to stringent FDR control over a massive 669-million-spectrum source, so results may not transfer to smaller or less carefully curated training sets.
- The paper notes the field 'would benefit from an exhaustive and rigorous benchmark comparison of this growing field of tools', so benchmark coverage of competing de novo tools is incomplete.
- No rigorous confidence-estimation method exists for de novo sequencing, which limits interpretation of raw prediction precision.
- Whether Casanovo outperforms standard database search in statistical power for a given DDA dataset remains an open question in the paper.

## Evidence

- [methods] Revised nine-species benchmark availability: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [discussion] MassIVE-KB training-data quality rationale: "Our experiments suggest that the carefully curated MassIVE-KB collection provides particularly good training data."
- [methods] Training set expansion for improved performance: "improved performance of Casanovo by training on the MassIVE-KB data"
- [readme] Casanovo sequencing capability: "Powered by a transformer neural network, Casanovo "translates" peaks in MS/MS spectra into amino acid sequences"
