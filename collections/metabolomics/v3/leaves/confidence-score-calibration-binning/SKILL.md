---
name: confidence-score-calibration-binning
description: Use when a frequency- or likelihood-based prioritization score (e.g.
  DeepMet sampling frequency given an exact mass within ±10 ppm) must be shown to
  be calibrated before being trusted for metabolite annotation or prospective discovery.
when_to_use_negative:
- The input scores come from MS/MS spectral matching rather than mass-only sampling-frequency
  prioritization; use the meta-learning annotation-integration approach instead.
- The model is being applied to evolutionarily distant metabolisms (e.g. plant or
  bacterial), where the human-metabolite-trained DeepMet prior and its calibration
  do not transfer and a bespoke model is required.
- No held-out ground-truth structures are available — calibration of correct vs. incorrect
  bins cannot be computed without known answers.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: DeepMet
  role: Source of the generated structures and sampling frequencies whose derived
    confidence scores are binned for calibration
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Computes exact monoisotopic masses of held-out metabolites to filter candidates
    at ±10 ppm before ranking and calibration
- name: Snakemake
  role: Runs the clm end-end workflow that produces the tabulated sampled-molecule
    files used in the calibration analysis
  repo: https://github.com/skinniderlab/CLM
- name: PyTorch
  role: Backend for the trained RNN language models that generated the candidate structures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:0a71b69b1ac4bd21eec0b0f7bcd36ebf0ff9911f4840e3aef95960146bf05cee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# confidence-score-calibration-binning

## Summary

Bin sampling-frequency-derived confidence scores for correct vs. incorrect metabolite annotations to verify that DeepMet's structure prioritization prior is well-calibrated. This validates that ranking accuracy (e.g. 29% top-1 in cross-validation) reflects trustworthy, usable confidence values rather than overconfident rankings.

## When to use

Use when a frequency- or likelihood-based prioritization score (e.g. DeepMet sampling frequency given an exact mass within ±10 ppm) must be shown to be calibrated before being trusted for metabolite annotation or prospective discovery.

## When NOT to use

- The input scores come from MS/MS spectral matching rather than mass-only sampling-frequency prioritization; use the meta-learning annotation-integration approach instead.
- The model is being applied to evolutionarily distant metabolisms (e.g. plant or bacterial), where the human-metabolite-trained DeepMet prior and its calibration do not transfer and a bespoke model is required.
- No held-out ground-truth structures are available — calibration of correct vs. incorrect bins cannot be computed without known answers.

## Inputs

- Tabulated sampled-molecule files from collect_tabulated_molecules / process_tabulated_molecules (molecular mass, formula, SMILES, sampling frequency per structure)
- Held-out metabolite list (cross-validation test folds and/or prospective HMDB v5.0 set) with SMILES/InChIKey identifiers
- Per-metabolite rank outcomes (rank of the correct structure among mass-filtered, frequency-sorted candidates)

## Outputs

- Calibration-bin statistics grouping confidence scores for correct vs. incorrect annotations
- top-1/top-3/top-10 accuracy values (targets: 29%/51%/72% cross-validation; 22.9%/61.7% prospective)
- evaluation_results.csv containing accuracies, rank distributions, AddCarbon-vs-DeepMet comparisons, and calibration-bin statistics

## How to apply

After ranking candidate structures for each held-out metabolite — obtained by computing exact monoisotopic masses with RDKit, filtering DeepMet-sampled structures (from the clm Snakemake pipeline's collect_tabulated_molecules / process_tabulated_molecules outputs) to those within ±10 ppm of the target mass, and sorting by decreasing sampling frequency — record whether the correct structure (matched by SMILES/InChIKey) was recovered. Then bin the sampling-frequency-derived confidence scores separately for correct vs. incorrect annotations and compare observed accuracy within each bin to the bin's average score, following the paper's calibration analysis. Aggregate results (top-1/top-3/top-10 accuracies, expected 29%/51%/72% in cross-validation and 22.9%/61.7% prospectively on HMDB v5.0) alongside calibration-bin statistics into evaluation_results.csv. If high-score bins contain proportionally more correct annotations in line with their assigned confidence, the prior is deemed well-calibrated and safe to use for prioritization without other analytical data.

## Related tools

- **DeepMet** (Source of the generated structures and sampling frequencies whose derived confidence scores are binned for calibration) — https://github.com/skinniderlab/CLM
- **RDKit** (Computes exact monoisotopic masses of held-out metabolites to filter candidates at ±10 ppm before ranking and calibration)
- **Snakemake** (Runs the clm end-end workflow that produces the tabulated sampled-molecule files used in the calibration analysis) — https://github.com/skinniderlab/CLM
- **PyTorch** (Backend for the trained RNN language models that generated the candidate structures)

## Examples

```
snakemake --use-conda --cores 8  # run clm workflow through collect_tabulated_molecules/process_tabulated_molecules, then bin confidence scores per write_structural_prior_CV logic
```

## Evaluation signals

- Cross-validation top-1/top-3/top-10 accuracies land at approximately 29%/51%/72%, matching the manuscript values
- Prospective (HMDB v5.0) top-1/top-10 accuracies land at approximately 22.9%/61.7%
- Within each confidence-score bin, the fraction of correct annotations increases monotonically with bin score (well-calibrated prior)
- DeepMet rank distributions outperform the AddCarbon-generated baseline database on the same mass-filtered candidates
- All accuracies, rank distributions, and calibration-bin statistics are present in evaluation_results.csv

## Limitations

- Calibration was established on human metabolites; transfer to evolutionarily distant metabolism (plant, bacterial) requires retraining and recalibration.
- The prior anticipates metabolites from known biosynthetic logic and has inherent limitations for metabolites from divergent, undiscovered biosynthetic routes.
- Well-calibrated prioritization holds even without other analytical data, but MS/MS spectra or retention-time prediction remain open problems and are not part of this calibration.

## Evidence
