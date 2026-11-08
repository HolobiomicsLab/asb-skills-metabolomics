---
name: denovo-peptide-sequencing-evaluation
description: Use when a de novo sequencing model (e.g. Casanovo) has been run on annotated
  MGF spectra and you need quantitative metrics — peptide-level AP, amino acid-level
  AP, and 30 ppm precursor mass accuracy — to validate or benchmark the predictions.
when_to_use_negative:
- Spectra are unannotated production data where no reference peptides exist to compute
  average precision — run de novo mode and use downstream confidence assignment instead
- The goal is database-search identification (Casanovo-DB mode, `casanovo db_search`)
  rather than evaluating de novo prediction quality
- Evaluating a database search engine like Tide+Percolator, where FDR-filtered PSM
  counts, not average precision, are the relevant metric
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Casanovo
  role: Runs de novo sequencing in sequence mode with --evaluate flag and beam search
    decoding to produce predictions and evaluation metrics
  repo: https://github.com/Noble-Lab/casanovo
- name: spectrum_utils
  role: Spectrum preprocessing used in the evaluation workflow
- name: PDV
  role: Graphical interface for inspecting Casanovo evaluation outputs
  repo: https://github.com/wenbostar/PDV/
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:a6eb7f6a63e33742bbf20d2ee4eb571ba1cc5a1c3df32b0fe8a7a46748c86ccb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - PDV
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# denovo-peptide-sequencing-evaluation

## Summary

Evaluate de novo peptide sequencing predictions (e.g. from Casanovo) by computing peptide-level and amino acid-level average precision and precursor mass accuracy against annotated benchmark spectra, then exporting predictions to mzTab.

## When to use

Use when a de novo sequencing model (e.g. Casanovo) has been run on annotated MGF spectra and you need quantitative metrics — peptide-level AP, amino acid-level AP, and 30 ppm precursor mass accuracy — to validate or benchmark the predictions.

## When NOT to use

- Spectra are unannotated production data where no reference peptides exist to compute average precision — run de novo mode and use downstream confidence assignment instead
- The goal is database-search identification (Casanovo-DB mode, `casanovo db_search`) rather than evaluating de novo prediction quality
- Evaluating a database search engine like Tide+Percolator, where FDR-filtered PSM counts, not average precision, are the relevant metric

## Inputs

- Annotated MGF files of a benchmark tandem MS dataset (e.g. nine-species benchmark, MassIVE MSV000090982)
- Pretrained Casanovo model weights (e.g. MassIVE-KB v1 30M PSM model, Zenodo 10.5281/zenodo.14973856)
- Casanovo configuration file with evaluation options

## Outputs

- Peptide-level average precision value
- Amino acid-level average precision value
- Fraction of predictions with precursor mass within 30 ppm of observed precursor
- mzTab file of spectrum predictions
- Results file recording evaluation metrics

## How to apply

Run the de novo tool (Casanovo) in sequence mode with the --evaluate flag on annotated MGF files, so predictions are generated with beam search decoding while being scored against the annotated peptide labels. Compute peptide-level and amino acid-level precision metrics (average precision), plus the fraction of predicted peptides whose precursor mass falls within 30 ppm of the annotated precursor m/z. Export the spectrum predictions to an mzTab file and record the metrics to a results file. Judge correctness by comparing the reproduced metrics to published values: peptide-level AP ≈ 0.95, amino acid-level AP ≈ 0.98 (Fig. 2b-c of the Casanovo paper), and ≥97% of predictions within 30 ppm precursor tolerance for the MassIVE-KB-trained model on the nine-species benchmark (MSV000090982).

## Related tools

- **Casanovo** (Runs de novo sequencing in sequence mode with --evaluate flag and beam search decoding to produce predictions and evaluation metrics) — https://github.com/Noble-Lab/casanovo
- **spectrum_utils** (Spectrum preprocessing used in the evaluation workflow)
- **PDV** (Graphical interface for inspecting Casanovo evaluation outputs) — https://github.com/wenbostar/PDV/

## Examples

```
casanovo sequence --evaluate --config casanovo.yaml benchmark.mgf
```

## Evaluation signals

- Reproduced peptide-level average precision approximates 0.95 for the MassIVE-KB-trained model on the nine-species benchmark
- At least 97% of predictions fall within 30 ppm of the observed precursor m/z
- Amino acid-level average precision approximates 0.98, matching reported Fig. 2b-c values
- mzTab export is produced containing the spectrum predictions
- Recorded metrics match published values within tolerance; discrepancies flagged for expert review

## Limitations

- No rigorous method for confidence estimation for de novo sequencing exists; the field lacks a validated confidence metric, complicating evaluation beyond labeled benchmarks
- The field lacks an exhaustive and rigorous benchmark comparison of the growing set of de novo sequencing tools
- The precursor mass filter is no longer applied in de novo mode, so mass-accuracy evaluation must rely on post-prediction precursor tolerance checks

## Evidence

- [other] Run Casanovo in sequence mode with the --evaluate flag on the benchmark MGF files, predicting peptides with beam search decoding using the pretrained model.: [evidence span withheld — anchor status: not_found]
- [other] Validation thresholds from the reproduction task card.: [evidence span withheld — anchor status: not_found]
- [abstract] Training set description from abstract.: "We train a Casanovo model from 30 million labeled spectra"
- [methods] Precursor mass filter behavior in de novo mode.: "The precursor mass filter is no longer applied in *de novo* mode"
- [methods] Benchmark dataset availability in methods.: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
