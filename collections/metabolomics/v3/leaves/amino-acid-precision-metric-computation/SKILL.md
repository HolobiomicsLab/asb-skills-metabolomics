---
name: amino-acid-precision-metric-computation
description: Use when evaluating de novo peptide sequencing predictions (e.g. from
  Casanovo) against annotated MGF benchmark spectra and you need peptide-level AP,
  amino acid-level AP, and precursor mass accuracy to compare against published values.
when_to_use_negative:
- 'The spectra are unannotated (no reference peptide sequences): average precision
  against ground truth cannot be computed — use de novo mode outputs and confidence
  estimation instead.'
- You need database-search scores (PSM q-values, target-decoy FDR) rather than de
  novo prediction accuracy against known annotations.
- The task is comparing statistical power to detect peptides versus a standard database
  search — the article flags this as an open question, not something this metric suite
  answers directly.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Casanovo
  role: de novo sequencing model run with --evaluate flag in sequence mode to produce
    predictions scored against annotated benchmark spectra
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:f5e8ed0ba79931cf051ae78860da1e1a9a6c47172df7d97fe2a2bba282c2c91f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# amino-acid-precision-metric-computation

## Summary

Computes peptide-level and amino acid-level average precision (AP) from de novo sequencing predictions evaluated against annotated benchmark spectra, alongside precursor mass accuracy (fraction of predictions within 30 ppm of the observed precursor m/z). This is the metric framework the Casanovo paper uses to quantify de novo sequencing quality on the nine-species benchmark.

## When to use

Use when evaluating de novo peptide sequencing predictions (e.g. from Casanovo) against annotated MGF benchmark spectra and you need peptide-level AP, amino acid-level AP, and precursor mass accuracy to compare against published values.

## When NOT to use

- The spectra are unannotated (no reference peptide sequences): average precision against ground truth cannot be computed — use de novo mode outputs and confidence estimation instead.
- You need database-search scores (PSM q-values, target-decoy FDR) rather than de novo prediction accuracy against known annotations.
- The task is comparing statistical power to detect peptides versus a standard database search — the article flags this as an open question, not something this metric suite answers directly.

## Inputs

- annotated MGF files of the nine-species benchmark (MassIVE dataset MSV000090982, doi:10.25345/C52V2CK8J)
- predicted peptides from Casanovo (or another de novo sequencing tool) run in sequence/evaluate mode with beam search decoding
- annotated precursor m/z values for each benchmark spectrum

## Outputs

- peptide-level average precision (target ≈ 0.95 for the MassIVE-KB-trained Casanovo model)
- amino acid-level average precision (target ≈ 0.98)
- fraction of predicted peptides with precursor mass within 30 ppm of the annotated precursor (target ≥ 97%)
- mzTab output file with spectrum predictions
- results file recording the evaluation metrics

## How to apply

Run the sequencing tool (e.g. Casanovo with the --evaluate flag in sequence mode) on the annotated MGF files of the nine-species benchmark (MassIVE MSV000090982) using a pretrained model, with beam search decoding enabled. For each spectrum, compare the predicted peptide to the annotated peptide: peptide-level AP counts a prediction as correct only when the full peptide sequence matches the annotation, while amino acid-level AP scores correctness at the individual amino acid position level. Additionally compute the fraction of predicted peptides whose precursor mass falls within 30 ppm of the annotated precursor m/z. Export predictions to an mzTab output file, record the metrics to a results file, and compare against published values (e.g. Fig. 2b-c: peptide-level AP ≈ 0.95, amino acid-level AP ≈ 0.98, ≥97% of predictions within 30 ppm for the MassIVE-KB-trained model).

## Related tools

- **Casanovo** (de novo sequencing model run with --evaluate flag in sequence mode to produce predictions scored against annotated benchmark spectra) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo sequence --evaluate --model casanovo_massivekb.pt benchmark.mgf --output benchmark_results.mztab
```

## Evaluation signals

- Reproduced peptide-level average precision approximates 0.95 for the MassIVE-KB-trained Casanovo model on the nine-species benchmark.
- Reproduced amino acid-level average precision approximates 0.98, matching the reported Fig. 2b-c values.
- At least 97% of predictions have m/z falling within 30 ppm of the observed precursor.
- Metrics are exported to a results file and predictions to a valid mzTab file.
- Results are comparable against the published Fig. 2b-c values from the Nature Communications paper (doi:10.1038/s41467-024-49731-x).

## Limitations

- Precursor mass accuracy is reported as the proportion within a fixed 30 ppm tolerance; spectra outside that window are counted as inaccurate regardless of sequence correctness.
- The article notes the field lacks a rigorous method for confidence estimation for de novo sequencing, which limits how AP values translate to per-spectrum confidence.
- There is no exhaustive, rigorous benchmark comparison across the growing set of de novo tools, so cross-tool AP comparisons may not be uniformly grounded.

## Evidence

- [results] Training on the MassIVE-KB dataset raises Casanovo's peptide-level average precision on the revised nine-species benchmark from 0.83 to 0.95: [evidence span withheld — anchor status: not_found]
- [methods] Compute peptide-level and amino acid-level precision metrics and the fraction of predicted peptides whose precursor mass falls within 30 ppm of the annotated precursor: [evidence span withheld — anchor status: not_found]
- [results] the reproduced peptide-level average precision should approximate 0.95, at least 97% of predictions should fall within 30 ppm precursor tolerance, and the amino acid-level average precision should approximate 0.98: [evidence span withheld — anchor status: not_found]
- [methods] Export spectrum predictions to mzTab files.: "Export spectrum predictions to mzTab files."
- [discussion] at this stage one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing: "at this stage one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing"
