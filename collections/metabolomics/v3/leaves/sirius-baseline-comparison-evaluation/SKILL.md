---
name: sirius-baseline-comparison-evaluation
description: Use when a new MS/MS annotation or property-prediction model (e.g. fingerprint
  retrieval, fluorine detection, similarity) must be evaluated head-to-head against
  the established SIRIUS-based method on the same labeled test spectra.
when_to_use_negative:
- The task is training or fine-tuning the model itself rather than evaluating it against
  an existing baseline (use the fine-tuning skill instead).
- The test spectra overlap structurally with the training library (e.g. shared Murcko
  scaffolds), which would invalidate the held-out comparison for both methods.
- Only a single method is of interest and no baseline (SIRIUS, MIST, MIST-CF) is available
  or applicable to the same spectra.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS
  role: Baseline annotation pipeline whose binary decision (e.g. fluorine presence)
    is compared against the candidate model on identical test spectra
- name: DreaMS
  role: Candidate model producing per-spectrum probability scores for the compared
    property (e.g. fluorine detection) from its fine-tuned checkpoint
  repo: https://github.com/pluskal-lab/DreaMS
- name: MSV000094528
  role: Source of the held-out test spectra (17,052 in-house fluorine-detection spectra)
    on which both methods are evaluated
- name: MoNA
  role: Training spectral library used to fine-tune the candidate; must be structurally
    disjoint from the test set
- name: NIST20
  role: Second training spectral library used for the candidate fine-tuning; excluded
    from the evaluation split
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:6dd07c09d2d2d4bd76e1aeeab577c4545267e0008c48754a7d43348e6aadd504
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - MSV000094528
  - MoNA
  - NIST20
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sirius-baseline-comparison-evaluation

## Summary

Benchmarks a spectral-property or annotation model against the SIRIUS pipeline by running both on a held-out test set of MS/MS spectra and reporting precision/recall (or equivalent) per method so the new model's advantage is quantified, e.g. DreaMS reaching precision 0.91 at recall 0.57 for fluorine detection versus SIRIUS not exceeding 0.51.

## When to use

Use when a new MS/MS annotation or property-prediction model (e.g. fingerprint retrieval, fluorine detection, similarity) must be evaluated head-to-head against the established SIRIUS-based method on the same labeled test spectra.

## When NOT to use

- The task is training or fine-tuning the model itself rather than evaluating it against an existing baseline (use the fine-tuning skill instead).
- The test spectra overlap structurally with the training library (e.g. shared Murcko scaffolds), which would invalidate the held-out comparison for both methods.
- Only a single method is of interest and no baseline (SIRIUS, MIST, MIST-CF) is available or applicable to the same spectra.

## Inputs

- Held-out test set of MS/MS spectra with known labels (e.g. fluorinated vs non-fluorinated), stored in the compact HDF5-based binary format or MGF
- Trained candidate model checkpoint (e.g. DreaMS Zenodo record 10997887)
- SIRIUS baseline installation (e.g. version 5.6.3) with equivalent task configuration

## Outputs

- Per-spectrum prediction table with method, score, and label (e.g. fluorine_detection_results.csv)
- Precision/recall/coverage metrics at swept thresholds for both candidate and SIRIUS baseline
- Comparison table highlighting the baseline precision ceiling (e.g. SIRIUS ≤ 0.51 vs candidate 0.91 precision)

## How to apply

Select a labeled test set of MS/MS spectra that was not used in training (in the DreaMS fluorine study, 17,052 in-house spectra released under MassIVE accession MSV000094528, disjoint from the MoNA and NIST20 training libraries). Run the candidate model end-to-end on these spectra to produce per-spectrum predictions with probability scores (DreaMS: 116M-parameter transformer initialized from the Zenodo record 10997887 pre-trained checkpoint, fine-tuned with focal binary cross-entropy α=0.8, γ=0.5). Run SIRIUS (e.g. version 5.6.3) as the baseline on the identical spectra, extracting the equivalent binary decision or score. Sweep the decision threshold on the candidate's scores to compute precision, recall, and coverage at a fixed precision target (e.g. 90% precision), then report both methods' metrics side by side and highlight the precision ceiling of the baseline. Correct application means both methods consumed the same test spectra, metrics are computed on identical subsets, and the baseline's known weakness (e.g. SIRIUS precision ≤ 0.51) is confirmed rather than assumed.

## Related tools

- **SIRIUS** (Baseline annotation pipeline whose binary decision (e.g. fluorine presence) is compared against the candidate model on identical test spectra)
- **DreaMS** (Candidate model producing per-spectrum probability scores for the compared property (e.g. fluorine detection) from its fine-tuned checkpoint) — https://github.com/pluskal-lab/DreaMS
- **MSV000094528** (Source of the held-out test spectra (17,052 in-house fluorine-detection spectra) on which both methods are evaluated)
- **MoNA** (Training spectral library used to fine-tune the candidate; must be structurally disjoint from the test set)
- **NIST20** (Second training spectral library used for the candidate fine-tuning; excluded from the evaluation split)

## Examples

```
python run_fluorine_eval.py --test-set MSV000094528 --candidate-checkpoint zenodo:10997887 --baseline sirius-5.6.3 --out fluorine_detection_results.csv
```

## Evaluation signals

- Both methods consumed the identical test spectra (same HDF5/MGF file, same 17,052-spectrum set) — any metric difference reflects the models, not the data.
- The baseline precision ceiling is empirically confirmed on this test set (SIRIUS does not exceed 0.51 precision), matching the reported figure before accepting the candidate's advantage.
- The candidate model reaches precision 0.91 at recall 0.57, and 95% of its predictions fall within the 90% precision coverage threshold.
- The test set is structurally disjoint from MoNA and NIST20 training molecules (Murcko-scaffold-disjoint split), so evaluation is genuinely held-out.

## Limitations

- The comparison is specific to the task and test distribution (e.g. fluorine detection on in-house spectra); baseline advantage may not generalize to other properties or other MS/MS data.
- Only positive ionization mode and singly charged precursor ions were covered; comparisons involving negative-mode or multiply charged spectra are out of scope of this evaluation setup.
- The candidate's fingerprint-prediction performance versus MIST is proof-of-concept only, and the CANOPUS benchmark is limited in size (8,000 spectra from 7,000 unique molecules), so cross-method generalization claims should be treated cautiously.

## Evidence

- [abstract] Baseline precision ceiling: "Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57"
- [discussion] Test set provenance and size: "The dataset is available under MassIVE accession number MSV000094528"
- [abstract] Candidate architecture and training detail: "a large self-supervised neural network (with 116 million parameters) trained directly on the repository-scale collection of raw experimental mass spectra"
- [abstract] Held-out split rationale: "the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules"
- [abstract] Coverage threshold definition: [evidence span withheld — anchor status: not_found]
