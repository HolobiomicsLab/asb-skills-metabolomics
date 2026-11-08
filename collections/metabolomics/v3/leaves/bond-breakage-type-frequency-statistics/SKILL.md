---
name: bond-breakage-type-frequency-statistics
description: Use when evaluating the interpretability of a fragmentation-graph predictor
  (e.g. ICEBERG) by checking whether its predicted bond breakages match chemistry
  intuition across a test set.
when_to_use_negative:
- Input spectra come from a black-box predictor that does not expose per-breakage
  bond annotations (e.g. NEIMS, MassFormer), since no bond-type statistics can be
  extracted.
- The goal is absolute spectrum prediction accuracy rather than interpretability of
  fragmentation pathways; frequency statistics do not measure cosine similarity or
  retrieval performance.
- Only a single molecule or very few molecules are available, making the frequency
  distribution too sparse to judge chemistry-level trends.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ICEBERG
  role: Generates fragmentation DAGs with per-breakage bond-type annotations that
    are aggregated into frequency statistics.
  repo: https://github.com/samgoldman97/ms-pred
- name: Python
  role: Runs ICEBERG inference and aggregation of bond-breakage frequencies via src/ms_pred/iceberg/predict_smis.py
    and custom aggregation code.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:a6ee90e61ab9022526daaaa8fe58a1e7be53ece20c62b0a74314bb426d0fddd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bond-breakage-type-frequency-statistics

## Summary

Aggregate the bond types broken across ICEBERG's predicted fragmentations on a held-out test set into a frequency table, revealing whether weaker heteroatom bonds (C-O, C-N) break more reliably than C-C bonds or complex ring breakages, and validating the model's chemical interpretability.

## When to use

Use when evaluating the interpretability of a fragmentation-graph predictor (e.g. ICEBERG) by checking whether its predicted bond breakages match chemistry intuition across a test set.

## When NOT to use

- Input spectra come from a black-box predictor that does not expose per-breakage bond annotations (e.g. NEIMS, MassFormer), since no bond-type statistics can be extracted.
- The goal is absolute spectrum prediction accuracy rather than interpretability of fragmentation pathways; frequency statistics do not measure cosine similarity or retrieval performance.
- Only a single molecule or very few molecules are available, making the frequency distribution too sparse to judge chemistry-level trends.

## Inputs

- Pretrained ICEBERG generator checkpoint (gen_ckpt)
- Pretrained ICEBERG intensity checkpoint (inten_ckpt)
- Held-out test molecules from processed dataset labels (e.g. NIST20/NIST23 test split)

## Outputs

- bond_breakage_stats.csv with bond-type frequency columns
- Annotated per-peak records containing fragment identity and hydrogen-rearrangement fields

## How to apply

Run ICEBERG inference via src/ms_pred/iceberg/predict_smis.py with the pretrained gen_ckpt and inten_ckpt checkpoints on held-out test molecules (~1 CPU second per molecule), producing fragmentation DAGs annotated with per-breakage bond types. Aggregate the frequencies of each broken bond type (C-O, C-N, C-C, ring breakages) across all held-out molecules into a summary table written to bond_breakage_stats.csv. The rationale is that ICEBERG's physical-fragmentation grounding lets each predicted peak be traced to a concrete bond-breaking event, so the aggregate distribution directly tests whether the model prioritizes chemically weaker bonds. Validate by confirming the CSV exists with bond-type frequency columns and that each annotated record carries fragment and hydrogen-shift fields.

## Related tools

- **ICEBERG** (Generates fragmentation DAGs with per-breakage bond-type annotations that are aggregated into frequency statistics.) — https://github.com/samgoldman97/ms-pred
- **Python** (Runs ICEBERG inference and aggregation of bond-breakage frequencies via src/ms_pred/iceberg/predict_smis.py and custom aggregation code.)

## Examples

```
python src/ms_pred/iceberg/predict_smis.py --gen_ckpt gen_ckpt --inten_ckpt inten_ckpt
```

## Evaluation signals

- bond_breakage_stats.csv exists and contains bond-type frequency columns (machine-checkable via field_present checks).
- Each annotated record carries fragment and hydrogen-shift fields for the most intense peaks.
- Broken bond-type frequencies show weaker heteroatom bonds (C-O, C-N) breaking more reliably than C-C bonds and complex ring breakages, matching chemistry intuition.
- Cases where a -1H shift corresponds to neutral H2O loss are flagged for expert chemistry plausibility review.
- Inference throughput is on the order of 1 CPU second per molecule, confirming the expected cost profile.

## Limitations

- There are no guarantees that the generated fragment explanations are the correct physical explanations due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective.
- ICEBERG incurs a higher computational cost of roughly 1 CPU second per molecule, nearly an order of magnitude more than neural approaches like SCARF, limiting large-scale throughput.
- The skill relies on ICEBERG's specific DAG format; changes between releases (e.g. ms_pred.dag_pred becoming ms_pred.iceberg) were not recorded in the changelog.

## Evidence

- [discussion] Inspection of ICEBERG's predicted fragmentations shows expected bond-breaking patterns consistent with chemical intuition: weaker heteroatom bonds (C-O, C-N) are predicted to break more reliably than C-C bonds and complex ring breakages.: [evidence span withheld — anchor status: not_found]
- [discussion] Higher computational cost (~1 CPU second per molecule, ~10x SCARF) limits throughput: "this comes at a higher computational cost of roughly 1 CPU second per molecule, nearly an order of magnitude more than other neural approaches like SCARF"
- [discussion] No guarantee that generated fragment explanations are the correct physical explanations due to equivalent-mass atom and bond rearrangements and cosine-similarity training objective: "there are no guarantees that they are the correct physical explanations"
- [intro] ICEBERG is fully open-sourced with pretrained weights alongside other existing prediction baseline methods available on GitHub: "ICEBERG is fully open-sourced with pretrained weights alongside other existing prediction baseline methods available on GitHub"
