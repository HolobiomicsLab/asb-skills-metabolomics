---
name: peptide-precursor-mass-tolerance-assessment
description: Use when validating de novo sequencing predictions (e.g., Casanovo output
  on annotated MGF benchmark spectra) and you need a precursor mass-accuracy metric
  independent of peptide identity accuracy.
when_to_use_negative:
- Input spectra lack reliable precursor m/z annotations, since the ppm tolerance comparison
  requires an observed precursor to compare against
- The task is database searching (Casanovo-DB mode) rather than de novo mode, where
  the precursor mass filter is applied differently — note the precursor mass filter
  is no longer applied in de novo mode
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Casanovo
  role: De novo peptide sequencer run in sequence mode with --evaluate to produce
    predictions whose precursor mass accuracy is assessed
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:420c885e754f06ecb00cde89da69cc4b83650aec67421dffbf769e9ad344e5c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# peptide-precursor-mass-tolerance-assessment

## Summary

Evaluates de novo peptide sequencing predictions by computing the fraction of predicted peptides whose theoretical precursor mass falls within 30 ppm of the observed precursor m/z, serving as a mass-accuracy quality check alongside peptide-level average precision.

## When to use

Use when validating de novo sequencing predictions (e.g., Casanovo output on annotated MGF benchmark spectra) and you need a precursor mass-accuracy metric independent of peptide identity accuracy.

## When NOT to use

- Input spectra lack reliable precursor m/z annotations, since the ppm tolerance comparison requires an observed precursor to compare against
- The task is database searching (Casanovo-DB mode) rather than de novo mode, where the precursor mass filter is applied differently — note the precursor mass filter is no longer applied in de novo mode

## Inputs

- Annotated MGF files of MS/MS spectra with precursor m/z annotations (e.g., nine-species benchmark, MassIVE MSV000090982)
- Predicted peptide sequences with confidence scores from a de novo sequencing model (e.g., Casanovo beam search output)
- Pretrained model weights (e.g., Casanovo trained on 30 million MassIVE-KB PSMs, Zenodo 10.5281/zenodo.14973856)

## Outputs

- Fraction of predicted peptides with precursor mass within 30 ppm of the observed precursor
- Peptide-level and amino acid-level average precision metrics
- mzTab output file of predicted peptides
- Results file recording evaluation metrics

## How to apply

Run the de novo sequencer (Casanovo in sequence mode with the --evaluate flag, using beam search decoding) over annotated MGF files, then for each predicted peptide compute its theoretical precursor mass and compare it to the annotated observed precursor m/z. Count predictions whose mass error is within 30 ppm of the observed precursor and report this as a fraction alongside peptide-level and amino acid-level average precision. In the Casanovo nine-species benchmark reproduction, the MassIVE-KB-trained model achieves at least 97% of predictions within 30 ppm, which is the expected validation threshold. Export predictions to mzTab and record the metrics to a results file so they can be compared against published values (Fig. 2b-c).

## Related tools

- **Casanovo** (De novo peptide sequencer run in sequence mode with --evaluate to produce predictions whose precursor mass accuracy is assessed) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo sequence --evaluate --model MassIVE-KB_v1.ckpt benchmark_species.mgf --output results.mztab
```

## Evaluation signals

- At least 97% of predictions fall within 30 ppm of the observed precursor mass on the nine-species benchmark
- Peptide-level average precision approximates 0.95 for the MassIVE-KB-trained model
- Amino acid-level average precision approximates 0.98, matching reported Fig. 2b-c values
- Reproduced metrics are compared against published Fig. 2b-c values for agreement

## Limitations

- Precursor mass accuracy within 30 ppm does not by itself confirm correct peptide identification; it must be interpreted together with peptide-level average precision
- Benchmark values are specific to the revised nine-species benchmark and the MassIVE-KB-trained model; other datasets or models may show different fractions within tolerance

## Evidence

- [other] Compute peptide-level and amino acid-level precision metrics and the fraction of predicted peptides whose precursor mass falls within 30 ppm of the annotated precursor.: [evidence span withheld — anchor status: not_found]
- [other] Validation: the reproduced peptide-level average precision should approximate 0.95, at least 97% of predictions should fall within 30 ppm precursor tolerance: [evidence span withheld — anchor status: not_found]
- [other] Training on the MassIVE-KB dataset raises Casanovo's peptide-level average precision on the revised nine-species benchmark from 0.83 to 0.95: [evidence span withheld — anchor status: not_found]
- [methods] The precursor mass filter is no longer applied in de novo mode: "The precursor mass filter is no longer applied in *de novo* mode"
