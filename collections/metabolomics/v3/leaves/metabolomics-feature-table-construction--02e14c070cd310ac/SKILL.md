---
name: metabolomics-feature-table-construction
description: Use when raw mzML files from multiple public LC/MS untargeted metabolomics
  studies (HILIC positive mode, serum) must be turned into a single parameter-free,
  drift-corrected feature matrix of robust peak groups for downstream modeling.
when_to_use_negative:
- Input is already a feature/intensity table — mzLearn starts from raw mzML files
  and re-running detection would discard prior peak picking
- Data are acquired in modes other than HILIC positive mode — the current pre-trained
  model and robust peak set were developed specifically for HILIC positive mode
- The goal requires de-isotoping, MS/MS identification, or compound annotation as
  part of the feature table itself — mzLearn outputs are at the ion MS1 level only
  and downstream workflows must handle these steps
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzLearn
  role: Parameter-free MS1 signal detection and alignment run on each study's mzML
    files; iteratively learns internal thresholds and drift maps from synthetic QC
    samples
  repo: github.com/ReviveMed/mzEmbed
- name: Eclipse
  role: Cross-study subalignment step that aggregates detected features across the
    22 studies via graph-based grouping
- name: metabCombiner
  role: Fits retention-time alignment splines across studies to correct inter-study
    RT drift after Eclipse grouping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:382c877f6aeb638718e23b3a1fa6034f65c32ac8ee3f41fead82a9e083f77aa7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - Eclipse
  - metabCombiner
  - mzLearn
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-feature-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs cross-study LC/MS MS1 feature tables by running mzLearn's parameter-free signal detection on raw mzML files and aggregating features across studies into robustness-filtered peak groups (2,736 robust groups from 22 HILIC positive-mode serum studies).

## When to use

Use when raw mzML files from multiple public LC/MS untargeted metabolomics studies (HILIC positive mode, serum) must be turned into a single parameter-free, drift-corrected feature matrix of robust peak groups for downstream modeling.

## When NOT to use

- Input is already a feature/intensity table — mzLearn starts from raw mzML files and re-running detection would discard prior peak picking
- Data are acquired in modes other than HILIC positive mode — the current pre-trained model and robust peak set were developed specifically for HILIC positive mode
- The goal requires de-isotoping, MS/MS identification, or compound annotation as part of the feature table itself — mzLearn outputs are at the ion MS1 level only and downstream workflows must handle these steps

## Inputs

- Raw mzML files for the 22 public HILIC positive-mode LC/MS serum studies listed in Supplementary Table 5, downloaded from the MetaboLights repository
- Per-study mzLearn intensity matrices (features defined by median rt and m/z values with normalized intensities)

## Outputs

- Cross-study-aligned, robustness-filtered peak-group table containing 2,736 robust metabolite signals
- Standardized (z-scored) CSV feature matrix of floats with shape [n_samples, 2736], e.g. X_Pretrain_Discovery_Train.csv / X_Pretrain_Discovery_Val.csv / X_Pretrain_Test.csv

## How to apply

Run mzLearn's data-driven MS1 signal detection on each study's mzML files — it requires no user-set parameters, instead generating synthetic QC samples and iteratively learning internal thresholds plus retention-time and intensity drift maps. Apply standard scaling (z-score) to each per-study intensity matrix so features are comparable across studies. Perform cross-study subalignment with Eclipse, which aggregates features across studies via graph-based grouping, then fit retention-time alignment splines with metabCombiner to correct inter-study RT drift. Finally filter aligned peak groups by robustness score to retain only signals reproducible across studies; on the 22-study pretraining set (20,548 blood samples) this yields 2,736 robust peak groups. Export the result as a CSV of floats shaped [n_samples, 2736] (first column = row index), which is the required input for mzEmbed VAE pretraining.

## Related tools

- **mzLearn** (Parameter-free MS1 signal detection and alignment run on each study's mzML files; iteratively learns internal thresholds and drift maps from synthetic QC samples) — github.com/ReviveMed/mzEmbed
- **Eclipse** (Cross-study subalignment step that aggregates detected features across the 22 studies via graph-based grouping)
- **metabCombiner** (Fits retention-time alignment splines across studies to correct inter-study RT drift after Eclipse grouping)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Final count of robust peak groups after robustness-score filtering equals 2,736, matching Supplementary Fig. 6
- Output feature matrix is a CSV of floats with shape [n_samples, 2736] and parses with pandas.read_csv(..., index_col=0)
- Each detected feature is defined by median rt and m/z values with normalized (standard-scaled) intensities across samples
- Per-study detection yield should exceed existing tools (mzLearn detects ~11,442 signals on average vs 7,100 XCMS and 4,655 ASARI, i.e. over 60% more on average)
- Alignment removed inter-study RT drift: retention-time splines fitted with metabCombiner across the Eclipse graph-based groupings

## Limitations

- Iterative learning is computationally heavy — analysis can take up to 2.5 days on a single machine, and parallel processing on multiple compute nodes is not yet implemented
- mzLearn results are at the ion MS1 level only; no de-isotoping or MS/MS identification is performed, and mapping ion features to MS/MS spectra requires downstream workflows
- No batch covariates or domain-invariant regularization are incorporated, and public repositories lack standardized QC samples to estimate instrument drift

## Evidence

- [readme] Output: A two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples.: [evidence span withheld — anchor status: not_found]
- [readme] Feature matrices: CSV of floats, shape [n_samples, 2736] (the robust mzLearn peak set).: [evidence span withheld — anchor status: not_found]
