---
name: predret-dataset-preprocessing
description: Use when loading PredRet retention time data from multiple LC systems
  via a config.json file and you need per-system cleaning (duplicate RTs, small-RT
  noise, >5% spread outliers) before order prediction or cross-system comparison.
when_to_use_negative:
- You have a single LC system's data and want absolute retention time prediction with
  SVR — the SVR baseline needs no cross-system pooling or order-based cleaning.
- The data are MS/MS spectra or candidate lists for IOKR scoring, not tabular retention
  times; preprocessing rules here target RT replicates only.
- You need retention times mapped to a common scale across instruments — this skill
  deliberately avoids scale mapping because it works with elution order instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0219
tools:
- name: pandas
  role: Loads and filters PredRet RT tables per system (keep lowest RT, drop >5% spread
    rows, drop small-RT molecules)
- name: numpy
  role: Numeric array support for RT threshold comparisons during filtering
- name: rcdk
  role: Used downstream for fingerprint calculation on the preprocessed molecules
  repo: https://github.com/rajarshi/cdkr
- name: rcdkTools
  role: Wrapper enabling MACCS counting fingerprint computation for the cleaned molecule
    sets
  repo: https://github.com/bachi55/rcdkTools
- name: evaluation_scenarios_main.py
  role: Consumes the preprocessed PredRet datasets for retention order prediction
    experiments
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:a82c37b6163374745be7f1622580ea6ca0625475713eb6f6c9ebe63ff63a499d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rajarshi/cdkr
  tools:
  - Python
  - evaluation_scenarios_main.py
  - numpy
  - pandas
  - rcdk
  - rcdkTools
  - scipy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# PredRet dataset preprocessing

## Summary

Cleans and harmonises PredRet retention time measurements per chromatographic system (e.g. Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old) so that heterogeneous LC datasets can be pooled for order-concordance analysis or RankSVM training without mapping retention times to a common scale.

## When to use

Use when loading PredRet retention time data from multiple LC systems via a config.json file and you need per-system cleaning (duplicate RTs, small-RT noise, >5% spread outliers) before order prediction or cross-system comparison.

## When NOT to use

- You have a single LC system's data and want absolute retention time prediction with SVR — the SVR baseline needs no cross-system pooling or order-based cleaning.
- The data are MS/MS spectra or candidate lists for IOKR scoring, not tabular retention times; preprocessing rules here target RT replicates only.
- You need retention times mapped to a common scale across instruments — this skill deliberately avoids scale mapping because it works with elution order instead.

## Inputs

- PredRet retention time measurement tables for chromatographic systems (e.g. Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old)
- Configuration file such as results/raw/PredRet/v2/config.json defining molecular features/kernels and the system set
- Molecule identity keys (e.g. InChI/SMILES identifiers) linking measurements across systems

## Outputs

- Cleaned per-system retention time tables (one row per molecule, lowest RT retained)
- Preprocessed datasets ready for fingerprint calculation (e.g. MACCS counting fingerprints) and RankSVM/SVR training
- Evaluation inputs for scenarios baseline_single, baseline_single_perc, all_on_one, all_on_one_perc

## How to apply

Load the PredRet measurements for each chromatographic system listed in the configuration (e.g. PredRet/v2/config.json) with pandas, processing one system at a time. For each system keep only the lowest retention time observed for a molecule and remove the molecule entirely when its lowest and largest retention time differ by more than 5%; this collapses replicate injections while retaining reproducible entries. Additionally remove molecules with very small retention times that do not interact with that chromatographic system, as they contribute spurious early-eluting pairs. Restrict measurements to those added before July 2015 when reproducing the published benchmark. The cleaned per-system tables are then ready for order-concordance scoring, Pearson/Spearman RT correlation, or as training input to the RankSVM (order) predictor in evaluation scenarios such as baseline_single or all_on_one.

## Related tools

- **pandas** (Loads and filters PredRet RT tables per system (keep lowest RT, drop >5% spread rows, drop small-RT molecules))
- **numpy** (Numeric array support for RT threshold comparisons during filtering)
- **rcdk** (Used downstream for fingerprint calculation on the preprocessed molecules) — https://github.com/rajarshi/cdkr
- **rcdkTools** (Wrapper enabling MACCS counting fingerprint computation for the cleaned molecule sets) — https://github.com/bachi55/rcdkTools
- **evaluation_scenarios_main.py** (Consumes the preprocessed PredRet datasets for retention order prediction experiments) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- After cleaning, no molecule retains more than one retention time per system (lowest RT only).
- For every retained molecule, max(RT) − min(RT) ≤ 5% of the value — verify the spread filter fired on replicates.
- No retained molecule has a 'very small retention time' that indicates non-interaction with the column for that system.
- All measurements used come from entries added before July 2015.
- Running evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json on cleaned data reproduces Table 3 results.

## Limitations

- Molecules whose lowest and largest RT differ by >5% are removed entirely, losing potentially informative compounds.
- Small-RT removal thresholds are heuristic; genuine early eluters interacting weakly with the column may be discarded.
- Preprocessing onlyif OpenBabel/obabel (>= 2.3.2) installed — only needed if raw data pre-processing must be repeated.

## Evidence

- [results] for each system we remove molecules with very small retention times from the dataset to avoid molecules that are not interacting with this chromatographic system: [evidence span withheld — anchor status: not_found]
