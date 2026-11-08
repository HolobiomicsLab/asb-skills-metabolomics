---
name: hydrogen-rearrangement-shift-assignment
description: Use when you have ICEBERG-generated fragmentation DAGs with per-peak
  hydrogen-shift annotations and want to verify that bond-breakage patterns and peak-to-fragment
  assignments align with chemical intuition.
when_to_use_negative:
- When the input is experimental spectra with no predicted fragmentation DAG — there
  are no per-peak hydrogen-shift annotations to audit.
- 'When throughput matters: ICEBERG costs roughly 1 CPU second per molecule, nearly
  an order of magnitude more than SCARF, so large-scale sweeps may warrant the faster
  formula-level models instead.'
- When you need proven ground-truth fragmentation explanations — the article states
  there are no guarantees that generated fragment explanations are the correct physical
  explanations.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0089
tools:
- name: ICEBERG
  role: Generates the fragmentation DAGs with per-breakage bond types and per-peak
    hydrogen-rearrangement annotations that this skill audits
  repo: https://github.com/samgoldman97/ms-pred
- name: Python
  role: Runs predict_smis.py inference and aggregates bond-breakage statistics into
    CSV for expert review
- name: MAGMa
  role: Bootstrapped to construct the fragmentation trees on which ICEBERG's generation
    model is trained, indirectly shaping the annotations under review
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:da400e8678ebff5e8eebe990e5f989924e8d4122624a04cdfa6e7899458e0052
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - MAGMa
  - Python
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hydrogen-rearrangement-shift-assignment

## Summary

Interpret ICEBERG's per-peak fragment attributions by extracting annotated hydrogen-rearrangement fields for intense peaks and auditing whether shifts like -1H correspond to chemically intuitive neutral losses (e.g. H2O), alongside aggregating predicted broken-bond types.

## When to use

Use when you have ICEBERG-generated fragmentation DAGs with per-peak hydrogen-shift annotations and want to verify that bond-breakage patterns and peak-to-fragment assignments align with chemical intuition.

## When NOT to use

- When the input is experimental spectra with no predicted fragmentation DAG — there are no per-peak hydrogen-shift annotations to audit.
- When throughput matters: ICEBERG costs roughly 1 CPU second per molecule, nearly an order of magnitude more than SCARF, so large-scale sweeps may warrant the faster formula-level models instead.
- When you need proven ground-truth fragmentation explanations — the article states there are no guarantees that generated fragment explanations are the correct physical explanations.

## Inputs

- Pretrained ICEBERG generator checkpoint (gen_ckpt)
- Pretrained ICEBERG intensity checkpoint (inten_ckpt)
- Held-out test molecules with SMILES/InChI from NIST20/NIST23 processed labels
- Configured elucidation YAML (e.g., iceberg_elucidation.yaml) pointing to checkpoint paths

## Outputs

- Per-peak fragment attribution records with fragment identity and hydrogen-rearrangement shift fields
- Aggregated bond-breakage frequency table (bond_breakage_stats.csv)
- Annotated examples of intense peaks flagged for -1H / H2O-loss correspondence review

## How to apply

Load the pretrained ICEBERG generator (gen_ckpt) and intensity (inten_ckpt) checkpoints and run inference over held-out test molecules (e.g., NIST20/NIST23 splits) via predict_smis.py, which produces fragmentation DAGs annotated with per-breakage bond types and per-peak hydrogen-rearrangement fields (~1 CPU second per molecule). Aggregate the frequency of broken bond types across molecules into a summary table; the article reports weaker heteroatom bonds (C-O, C-N) breaking more reliably than C-C bonds or complex ring breakages, so deviations from that ordering flag annotation or model issues. For the most intense peaks, retain the fragment identity and hydrogen-shift fields, then flag cases where a -1H shift corresponds to a neutral H2O loss and judge that correspondence by expert chemistry review. Note the authors caution there are no guarantees that generated fragment explanations are the correct physical explanations, because equivalent-mass atom and bond rearrangements and the vectorized-cosine-similarity training objective can admit alternatives.

## Related tools

- **ICEBERG** (Generates the fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement annotations that this skill audits) — https://github.com/samgoldman97/ms-pred
- **Python** (Runs predict_smis.py inference and aggregates bond-breakage statistics into CSV for expert review)
- **MAGMa** (Bootstrapped to construct the fragmentation trees on which ICEBERG's generation model is trained, indirectly shaping the annotations under review)

## Examples

```
python src/ms_pred/iceberg/predict_smis.py --gen-ckpt gen_ckpt --inten-ckpt inten_ckpt --dataset test
```

## Evaluation signals

- bond_breakage_stats.csv exists with bond-type frequency columns covering C-O, C-N, C-C and ring-breakage classes
- Every annotated record for intense peaks carries non-empty fragment and hydrogen-shift fields (field_present checks)
- Broken-bond frequency ordering matches chemical intuition: weaker heteroatom bonds (C-O, C-N) appear more reliably than C-C or complex ring breakages
- Flagged -1H peaks where hydrogen loss equals neutral H2O loss are reviewed and accepted by an expert chemist
- Inference runtime stays near ~1 CPU second per molecule on the held-out set

## Limitations

- No guarantee that the generated fragment explanations are the correct physical explanations, due to equivalent-mass atom and bond rearrangements and the vectorized cosine-similarity training objective.
- Higher computational cost of roughly 1 CPU second per molecule, nearly an order of magnitude more than other neural approaches like SCARF.
- Chemistry plausibility of the -1H/H2O-loss correspondence requires expert review rather than automated validation.

## Evidence

- [results] weaker heteroatom bonds break more reliably: [evidence span withheld — anchor status: not_found]
