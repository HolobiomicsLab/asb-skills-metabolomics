---
name: cross-species-benchmark-comparison
description: Use when you have de novo sequencing predictions on annotated MS/MS spectra
  from several species or heterogeneous samples and need a standardized peptide- and
  amino-acid-level accuracy comparison across models or training datasets.
when_to_use_negative:
- Input spectra are unlabeled or from a single homogeneous species with no ground-truth
  peptide annotations — precision cannot be computed without annotated MGF ground
  truth.
- The goal is sequence database searching with a known protein database (Casanovo-DB
  mode), not de novo accuracy benchmarking across tools or training sets.
- Evaluating spectra containing modifications absent from the benchmark's annotation
  vocabulary (e.g. non-enzymatic immunopeptidomics PTMs) — use the fine-tuned non-enzymatic
  Casanovo model instead.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Casanovo
  role: De novo sequencing model run in sequence mode with --evaluate to generate
    benchmark predictions on annotated MGF files
  repo: https://github.com/Noble-Lab/casanovo
- name: DeepNovo
  role: Competing de novo sequencing baseline for cross-species performance comparison
  repo: https://github.com/nh2tran/DeepNovo
- name: PDV
  role: Graphical interface for inspecting Casanovo benchmark outputs
  repo: https://github.com/wenbostar/PDV/
- name: limelight-import-casanovo
  role: Converts Casanovo mzTab benchmark results to Limelight XML for downstream
    analysis
  repo: https://github.com/yeastrc/limelight-import-casanovo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:addf74cf287c668fd1a79b4daf1bab353d9aab25416c0dbb015e2005ab5492c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - DeepNovo
  - PDV
  - limelight-import-casanovo
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-species-benchmark-comparison

## Summary

Evaluate de novo peptide sequencing models on a multi-species annotated MGF benchmark (MassIVE dataset MSV000090982), computing peptide-level and amino acid-level average precision plus precursor mass accuracy so that performance can be compared across tools and training sets. The published benchmark shows a MassIVE-KB-trained Casanovo reaching peptide-level AP ≈0.95 on nine species.

## When to use

Use when you have de novo sequencing predictions on annotated MS/MS spectra from several species or heterogeneous samples and need a standardized peptide- and amino-acid-level accuracy comparison across models or training datasets.

## When NOT to use

- Input spectra are unlabeled or from a single homogeneous species with no ground-truth peptide annotations — precision cannot be computed without annotated MGF ground truth.
- The goal is sequence database searching with a known protein database (Casanovo-DB mode), not de novo accuracy benchmarking across tools or training sets.
- Evaluating spectra containing modifications absent from the benchmark's annotation vocabulary (e.g. non-enzymatic immunopeptidomics PTMs) — use the fine-tuned non-enzymatic Casanovo model instead.

## Inputs

- Annotated MGF files of the nine-species benchmark (MassIVE MSV000090982)
- Pretrained Casanovo model weights trained on MassIVE-KB 30 million PSMs (Zenodo 10.5281/zenodo.14973856)
- MassIVE-KB training/validation/test splits (Zenodo 10.5281/zenodo.14967861)

## Outputs

- Peptide-level and amino acid-level average precision metrics
- Fraction of predictions within 30 ppm of the observed precursor m/z
- mzTab output file of predicted peptides
- Results file with recorded evaluation metrics for cross-tool comparison

## How to apply

Download the annotated MGF files of the nine-species benchmark from MassIVE (identifier MSV000090982) and run Casanovo in sequence mode with the --evaluate flag, using pretrained weights trained on 30 million MassIVE-KB PSMs (Zenodo 10.5281/zenodo.14973856) with beam search decoding. Compute peptide-level and amino acid-level precision metrics, and the fraction of predicted peptides whose precursor mass falls within 30 ppm of the annotated precursor, using the same splits as the paper where available (training/validation/test splits on Zenodo 10.5281/zenodo.14967861). Export predictions to mzTab for downstream comparison against other de novo tools such as DeepNovo, Novor, or PointNovo, and record the metrics to a results file. Validate by reproducing the published reference values: peptide-level average precision ≈0.95, ≥97% of predictions within 30 ppm precursor tolerance, and amino acid-level average precision ≈0.98 (Fig. 2b-c).

## Related tools

- **Casanovo** (De novo sequencing model run in sequence mode with --evaluate to generate benchmark predictions on annotated MGF files) — https://github.com/Noble-Lab/casanovo
- **DeepNovo** (Competing de novo sequencing baseline for cross-species performance comparison) — https://github.com/nh2tran/DeepNovo
- **PDV** (Graphical interface for inspecting Casanovo benchmark outputs) — https://github.com/wenbostar/PDV/
- **limelight-import-casanovo** (Converts Casanovo mzTab benchmark results to Limelight XML for downstream analysis) — https://github.com/yeastrc/limelight-import-casanovo

## Examples

```
casanovo sequence --evaluate --output benchmark_predictions.mzTab --model MassIVE-KB_v1.ckpt nine_species_benchmark.mgf
```

## Evaluation signals

- Reproduced peptide-level average precision approximates 0.95 on the nine-species benchmark
- At least 97% of predictions fall within 30 ppm precursor tolerance
- Amino acid-level average precision approximates 0.98, matching Fig. 2b-c
- mzTab output parses cleanly and predictions span the expected number of benchmark spectra
- Metrics match published Fig. 2b-c values when compared against reference runs

## Limitations

- The field lacks an exhaustive and rigorous benchmark comparison of the growing set of de novo sequencing tools, so results may not generalize beyond the nine species tested.
- No rigorous method for confidence estimation for de novo sequencing exists, making precision-based evaluation the primary but imperfect accuracy measure.
- Open question whether Casanovo outperforms standard database search in statistical power to detect peptides for a given DDA dataset — benchmark AP does not answer that.

## Evidence

- [results] Reproduce MassIVE-KB-trained Casanovo results on the revised nine-species benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark data provenance: "available as annotated MGF files on MassIVE with [dataset identifier MSV000090982]"
- [intro] Training set expansion: "We expand our training set to use 30 million confident PSMs from the MassIVE-KB spectral library23, and we add a beam search decoding procedure to predict the best peptide for each MS/MS spectrum."
- [abstract] Abstract-level claim: "demonstrate that the model outperforms several state-of-the-art methods on a cross-species benchmark dataset"
- [discussion] Tool limitation caveat: "Clearly, the field would benefit from an exhaustive and rigorous benchmark comparison of this growing field of tools."
- [readme] README tool role: "Casanovo is a state-of-the-art deep learning tool designed for _de novo_ peptide sequencing."
