---
name: retention-order-integrated-metabolite-identification
description: Use when you already have per-spectrum scored candidate molecule lists
  from an MS/MS predictor (e.g. IOKR) plus retention times for a complete LC-MS/MS
  run, and want to jointly identify the whole metabolite set using retention order
  constraints.
when_to_use_negative:
- You only have a single MS/MS spectrum to identify (no retention-time ordering information
  across the run) — the layered graph and rank-conflict penalty are undefined without
  ordered layers
- Retention times are unavailable for the target chromatographic system and no measurements
  from other LC systems exist to train the RankSVM order predictor
- You want standalone retention-time prediction rather than retention-order-based
  joint identification — the paper notes SVR-based retention time prediction is limited
  in its capacity to use data from other chromatographic systems, unlike RankSVM
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0153
tools:
- name: IOKR
  role: MS/MS-based predictor producing the scored candidate molecule lists that seed
    the layered graph
- name: scikit-learn (sklearn)
  role: RankSVM order predictor used to score retention-order conflicts via the molecular
    feature map
- name: numpy
  role: Numerical implementation of edge weights and the dynamic programming shortest-path
    pass
- name: networkx
  role: Layered directed graph construction for the candidate-sequence shortest-path
    problem (required >= 2.0)
- name: rcdkTools
  role: Computation of MACCS counting fingerprints used as the molecular output representation
  repo: https://github.com/bachi55/rcdkTools
- name: rcdk
  role: Fingerprint calculation underlying the molecular feature map
  repo: https://github.com/rajarshi/cdkr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:a635d93e12b7957fffc09a298e5417ba514bd3a50b213ded562067ed90d79b6a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bachi55/rcdkTools
  tools:
  - IOKR
  - Python and
  - R
  - joblib
  - networkx
  - numpy
  - pandas
  - rcdk
  - rcdkTools
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 20
---

# retention-order-integrated-metabolite-identification

## Summary

Combines IOKR MS/MS-based candidate scores with RankSVM-predicted retention orders by solving a shortest-path problem over a layered directed graph of candidate structures, improving metabolite identification accuracy across a complete LC-MS/MS run (up to 24.8% at D=0.0075 in the PredRet LTSO setting).

## When to use

Use when you already have per-spectrum scored candidate molecule lists from an MS/MS predictor (e.g. IOKR) plus retention times for a complete LC-MS/MS run, and want to jointly identify the whole metabolite set using retention order constraints.

## When NOT to use

- You only have a single MS/MS spectrum to identify (no retention-time ordering information across the run) — the layered graph and rank-conflict penalty are undefined without ordered layers
- Retention times are unavailable for the target chromatographic system and no measurements from other LC systems exist to train the RankSVM order predictor
- You want standalone retention-time prediction rather than retention-order-based joint identification — the paper notes SVR-based retention time prediction is limited in its capacity to use data from other chromatographic systems, unlike RankSVM

## Inputs

- Per-spectrum scored candidate molecule lists produced by IOKR (input output kernel regression) for each MS/MS spectrum in the run
- Trained RankSVM order predictor: weight vector w and molecular feature map (MACCS counting fingerprints, minmax kernel)
- Retention times of the measured MS/MS spectra in the LC-MS/MS run (e.g. PredRet data from Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old)
- Experiment configuration JSON (e.g. results/raw/PredRet/v2/config.json) specifying molecular representations and kernels

## Outputs

- Optimal sequence of candidate molecular structures in retention order (one identification per measured MS/MS spectrum)
- Identification accuracy per D value and training variant (e.g. identification_accuracy_vs_D.csv: average percentage of correctly identified molecular structures across the run)

## How to apply

Load the IOKR-scored candidate lists together with the RankSVM weight vector w and the molecular feature map (MACCS counting fingerprints with minmax kernel) from the experiment config (e.g. results/raw/PredRet/v2/config.json). Build a layered directed graph with one layer per measured MS/MS spectrum sorted by increasing retention time, candidates sorted by decreasing IOKR score, and directed edges between every node pair in consecutive layers. Assign each edge the weight d = -y(r,s) + D * max(0, wT(phi(m(i,j)) - phi(m(r,s)))), where the hinge term penalizes only rank conflicts with observed retention-time order and is zero when observed retention times of layers i and r are equal. Run the dynamic programming shortest-path forward pass (O(NK^2) for N layers and maximum layer size K) to retrieve the optimal retention-ordered sequence of candidate structures. Sweep the balance constant D over the repository's evaluated range d_lower=0 to d_upper=16; D=0 reproduces the IOKR-only baseline, and the PredRet LTSO (all_on_one) scenario should be repeated for the Target / Others / Others & target training variants.

## Related tools

- **IOKR** (MS/MS-based predictor producing the scored candidate molecule lists that seed the layered graph)
- **scikit-learn (sklearn)** (RankSVM order predictor used to score retention-order conflicts via the molecular feature map)
- **numpy** (Numerical implementation of edge weights and the dynamic programming shortest-path pass)
- **networkx** (Layered directed graph construction for the candidate-sequence shortest-path problem (required >= 2.0))
- **rcdkTools** (Computation of MACCS counting fingerprints used as the molecular output representation) — https://github.com/bachi55/rcdkTools
- **rcdk** (Fingerprint calculation underlying the molecular feature map) — https://github.com/rajarshi/cdkr

## Examples

```
python src/evaluation_scenarios_main.py ranksvm all_on_one 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Setting D=0 must reproduce the IOKR-only identification baseline (average percentage of correctly identified molecular structures over the full run)
- The integrated results must reproduce the reported qualitative trend: combining retention order predictions with MS/MS-based scores improves accuracy over the IOKR-only baseline, as validated against the ECCB2018.Rmd report
- Up to 24.8% improvement at D=0.0075 when RankSVM is trained on Others & target retention times (LTSO scenario) is the reference magnitude
- Results directory structure must match ranksvm_slacktype=on_pairs / allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph with the chosen fingerprint and kernel (e.g. maccsCount_f2dcf0b3/minmax)
- The hinge term must evaluate to zero for edge pairs whose observed layer retention times are equal (no penalty for co-eluting layers)

## Limitations

- Improvement depends on training the RankSVM on retention measurements from the target system and/or other chromatographic systems; training-set choice (Target / Others / Others & target) materially changes accuracy
- The method requires a complete LC-MS/MS run with retention times for all spectra; it cannot improve identification of isolated spectra
- Code was tested only with Python 3.5/3.6 (scipy >= 0.19.1, numpy >= 1.13.1, pandas >= 0.20.3, sklearn >= 0.19.0, networkx >= 2.0) and R 3.4 for pre-processing/reporting
- Repeating data pre-processing additionally requires OpenBabel >= 2.3.2 (obabel CLI) plus the obabel2R and rcdk R packages
- Molecules with very small retention times (non-interacting with the chromatographic system) and molecules with >5% retention-time spread are removed during pre-processing, affecting which spectra can be integrated

## Evidence

- [methods] Algorithm complexity: [evidence span withheld — anchor status: not_found]
