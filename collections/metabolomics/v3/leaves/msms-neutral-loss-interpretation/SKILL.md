---
name: msms-neutral-loss-interpretation
description: Use when interpreting ICEBERG fragment attribution outputs — e.g., after
  running predict_smis.py on held-out molecules — to check whether breakage frequencies
  and hydrogen shifts match chemistry intuition (C-O/C-N favored over C-C; -1H ↔ H2O
  loss).
when_to_use_negative:
- When the goal is ground-truth assignment of which atom/bond rearrangement physically
  produced a peak — ICEBERG's cosine-similarity training and equivalent-mass rearrangements
  mean attributions may not be the correct physical explanations
- When throughput matters more than fragment-level interpretability — ICEBERG costs
  ~1 CPU second per molecule, nearly an order of magnitude more than SCARF, which
  predicts only at the formula level and yields no bond-breakage or hydrogen-shift
  attributions
- When working with the retired NPLIB1 dataset on the main branch — use the legacy
  iceberg_1.0 branch instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: ICEBERG
  role: Generate fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement
    annotations for held-out molecules
  repo: https://github.com/samgoldman97/ms-pred
- name: ICEBERG (ms-pred repo)
  role: Run src/ms_pred/iceberg/predict_smis.py inference and aggregate bond-type
    breakage statistics from its outputs
  repo: https://github.com/coleygroup/ms-pred
- name: Python
  role: Execute predict_smis.py and post-processing scripts that build bond_breakage_stats.csv
    and extract per-peak annotated records
- name: MAGMa
  role: Bootstrapped to construct the fragmentation trees on which ICEBERG's generator
    was trained, defining the DAG/bond-annotation schema being interpreted
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:55636b7436e3095c5bd83b1df5768ae3258efc433a87522339364a5fe3237818
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - ICEBERG (ms-pred repo)
  - MAGMa
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# msms-neutral-loss-interpretation

## Summary

Inspect ICEBERG-predicted fragmentations to aggregate which bond types are broken across a held-out set (e.g., NIST20/NIST23 test splits) and verify whether per-peak hydrogen-rearrangement annotations (-1H shifts) correspond to chemically intuitive neutral losses such as H2O, as reported for Figure 4.

## When to use

Use when interpreting ICEBERG fragment attribution outputs — e.g., after running predict_smis.py on held-out molecules — to check whether breakage frequencies and hydrogen shifts match chemistry intuition (C-O/C-N favored over C-C; -1H ↔ H2O loss).

## When NOT to use

- When the goal is ground-truth assignment of which atom/bond rearrangement physically produced a peak — ICEBERG's cosine-similarity training and equivalent-mass rearrangements mean attributions may not be the correct physical explanations
- When throughput matters more than fragment-level interpretability — ICEBERG costs ~1 CPU second per molecule, nearly an order of magnitude more than SCARF, which predicts only at the formula level and yields no bond-breakage or hydrogen-shift attributions
- When working with the retired NPLIB1 dataset on the main branch — use the legacy iceberg_1.0 branch instead

## Inputs

- Pretrained ICEBERG generator checkpoint (gen_ckpt) and intensity checkpoint (inten_ckpt) — e.g., NIST'23 pretrained weights from the iceberg_2.0 branch or MassSpecGym-trained weights
- Held-out test molecules with structures from the processed dataset labels (NIST20/NIST23 test split; NPLIB1 via legacy iceberg_1.0 branch)
- Raw NIST'20/23/26 SDF exports processed via reformat_nist_lcmsms_sdf.py and subformulae assigned via assign_subformulae.py
- Open-source alternative: MassSpecGym 1.5 TSV processed with create_msg_simulation_dataset.py --strategy msg_all

## Outputs

- bond_breakage_stats.csv — aggregated frequencies of broken bond types (C-O, C-N, C-C, ring breakages) across all held-out molecules
- Per-peak annotated records for the most intense peaks containing fragment identity and hydrogen-rearrangement fields, with flags for -1H shifts corresponding to neutral H2O loss
- Fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement annotations from predict_smis.py

## How to apply

Load the pretrained ICEBERG generator (gen_ckpt) and intensity (inten_ckpt) checkpoints and run inference via src/ms_pred/iceberg/predict_smis.py on the held-out test molecules from the processed dataset labels (NIST20/NIST23 splits), producing fragmentation DAGs annotated with per-breakage bond types and per-peak hydrogen-rearrangement fields (~1 CPU second per molecule). Aggregate broken bond-type frequencies across all held-out molecules into bond_breakage_stats.csv and inspect whether weaker heteroatom bonds (C-O, C-N) dominate over C-C and complex ring breakages. Separately, extract annotated records for the most intense peaks, retaining fragment identity and hydrogen-shift fields, and flag cases where a -1H shift corresponds to neutral H2O loss. Validation: confirm bond_breakage_stats.csv exists with bond-type frequency columns and every annotated record carries fragment and shift fields (machine-checkable), plus expert review for chemistry plausibility of the -1H/H2O correspondence. Note the authors caution that predicted attributions are not guaranteed to be the correct physical explanations, so interpretation must be framed as consistency-with-intuition rather than ground-truth assignment.

## Related tools

- **ICEBERG** (Generate fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement annotations for held-out molecules) — https://github.com/samgoldman97/ms-pred
- **ICEBERG (ms-pred repo)** (Run src/ms_pred/iceberg/predict_smis.py inference and aggregate bond-type breakage statistics from its outputs) — https://github.com/coleygroup/ms-pred
- **Python** (Execute predict_smis.py and post-processing scripts that build bond_breakage_stats.csv and extract per-peak annotated records)
- **MAGMa** (Bootstrapped to construct the fragmentation trees on which ICEBERG's generator was trained, defining the DAG/bond-annotation schema being interpreted) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
python src/ms_pred/iceberg/predict_smis.py --gen_ckpt <gen_ckpt> --inten_ckpt <inten_ckpt> --dataset labels/nist20_test.json
```

## Evaluation signals

- bond_breakage_stats.csv exists and contains columns with per-bond-type breakage frequencies (C-O, C-N, C-C, ring breakage) summed over the held-out set
- Every extracted intense-peak record carries non-null fragment identity and hydrogen-shift fields (machine-checkable via field_present checks)
- C-O and C-N break frequencies exceed C-C frequencies, matching the expected heteroatom-bond susceptibility pattern
- Expert review confirms the flagged -1H hydrogen shifts plausibly correspond to neutral H2O losses on the most intense peaks
- Inference completes at roughly 1 CPU second per molecule on the held-out set, consistent with ICEBERG's reported runtime

## Limitations

- Predicted attributions carry no guarantee of being the correct physical explanations due to equivalent-mass atom/bond rearrangements and the cosine-similarity training objective
- Higher computational cost (~1 CPU second per molecule, ~10x SCARF) limits throughput
- Chemistry plausibility of -1H/H2O-loss correspondences must be judged by expert review, not automated checks alone
- Model naming changed between branches (ms_pred.dag_pred → ms_pred.iceberg) and was not recorded in the changelog; release 1.1.0 is unreleased

## Evidence

- [results] Inspect ICEBERG predicted fragmentations for chemistry-intuitive bond-breakage and hydrogen-shift attributions: [evidence span withheld — anchor status: not_found]
