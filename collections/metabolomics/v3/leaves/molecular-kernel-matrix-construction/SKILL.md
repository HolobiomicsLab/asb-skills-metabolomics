---
name: molecular-kernel-matrix-construction
description: 'Use when training a pairwise RankSVM on molecular structures for LC
  retention-order prediction: structures are given as InChI/MACCS representations
  and a kernel matrix between training molecules is required.'
when_to_use_negative:
- The goal is predicting absolute retention times rather than pairwise elution order
  — the paper uses SVR with per-system RT calibration for that, which cannot leverage
  heterogeneous LC systems the way the kernelized RankSVM can.
- The prediction task operates on MS/MS spectra rather than molecular structures —
  molecular candidate scoring is handled by IOKR, not by the molecular kernel matrix.
- Molecule representations are already provided as explicit feature vectors and a
  linear kernel suffices; building a MinMax/Tanimoto molecular kernel is unnecessary
  overhead.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rcdkTools
  role: R package that computes counting MACCS fingerprints via the CDK; provides
    the phi(m) feature maps whose inner products form K_m
  repo: https://github.com/bachi55/rcdkTools
- name: rcdk
  role: Base rcdk R package used for fingerprint calculation from molecular structures
    (InChI input)
  repo: https://github.com/rajarshi/cdkr
- name: retention_order_prediction (Python pipeline)
  role: Python code (scipy/numpy/sklearn) that assembles K_m and decomposes Q = A
    K_m A^T inside the RankSVM training; kernel choice read from config.json
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:b9d152128c781264afe2cdaf66ab307848ffe80365d1c7ad0f4c9a35662708a7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bachi55/rcdkTools
  tools:
  - Python
  - joblib
  - numpy
  - pandas
  - rcdk
  - rcdkTools
  - retention_order_prediction (Python pipeline)
  - scikit-learn (sklearn)
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# molecular-kernel-matrix-construction

## Summary

Constructs the molecular kernel matrix K_m with entries [K_m]_ij = phi(m_i)^T phi(m_j) from MACCS fingerprints, using the MinMax kernel for counting fingerprints and the Tanimoto kernel for binary fingerprints. This kernel is the molecular side of the RankSVM retention-order pipeline and is decomposed as Q = A K_m A^T so the full pairwise kernel never needs materializing.

## When to use

Use when training a pairwise RankSVM on molecular structures for LC retention-order prediction: structures are given as InChI/MACCS representations and a kernel matrix between training molecules is required.

## When NOT to use

- The goal is predicting absolute retention times rather than pairwise elution order — the paper uses SVR with per-system RT calibration for that, which cannot leverage heterogeneous LC systems the way the kernelized RankSVM can.
- The prediction task operates on MS/MS spectra rather than molecular structures — molecular candidate scoring is handled by IOKR, not by the molecular kernel matrix.
- Molecule representations are already provided as explicit feature vectors and a linear kernel suffices; building a MinMax/Tanimoto molecular kernel is unnecessary overhead.

## Inputs

- InChI representations of molecular structures (e.g. from the PredRet dataset, configured via results/raw/PredRet/v2/config.json)
- MACCS counting fingerprints (maccsCount_f2dcf0b3) or binary MACCS fingerprints (maccs) computed via rcdkTools/CDK
- Experiment config JSON specifying molecule_representation: kernel (minmax|tanimoto), predictor, feature_scaler

## Outputs

- Molecular kernel matrix K_m with [K_m]_ij = phi(m_i)^T phi(m_j)
- Kernel-derived pairwise matrix Q = A K_m A^T consumed by the RankSVM dual optimization (0 <= alpha_ij <= C)

## How to apply

Compute MACCS molecular fingerprints for all training molecules using the CDK via the rcdkTools R package (a modified cdkr fork with branch support_cdk_substructure_fps), taking InChI representations of the molecular structures as input. Choose the kernel type according to the fingerprint flavor: counting MACCS fingerprints (maccsCount) pair with the MinMax kernel, while binary MACCS fingerprints (maccs) pair with the Tanimoto kernel — this choice is configured in the experiment config JSON under molecule_representation (kernel: minmax vs tanimoto). Assemble the n x n molecular kernel matrix K_m with [K_m]_ij = phi(m_i)^T phi(m_j). In the RankSVM training stage, decompose the pairwise quadratic matrix as Q = A K_m A^T so that only K_m (not the full pairwise kernel) must be materialized. Feature scaling is set to noscaling for these kernel-based representations.

## Related tools

- **rcdkTools** (R package that computes counting MACCS fingerprints via the CDK; provides the phi(m) feature maps whose inner products form K_m) — https://github.com/bachi55/rcdkTools
- **rcdk** (Base rcdk R package used for fingerprint calculation from molecular structures (InChI input)) — https://github.com/rajarshi/cdkr
- **retention_order_prediction (Python pipeline)** (Python code (scipy/numpy/sklearn) that assembles K_m and decomposes Q = A K_m A^T inside the RankSVM training; kernel choice read from config.json) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- K_m is square with dimension equal to the number of training molecules and is symmetric; diagonal entries equal each molecule's self-similarity (1 for Tanimoto on binary fingerprints).
- Kernel type matches fingerprint type: MinMax kernel selected with maccsCount predictor, Tanimoto kernel selected with binary maccs predictor (config.json molecule_representation).
- feature_scaler is 'noscaling' — applying a scaler to kernelized representations signals misconfiguration.
- Downstream RankSVM pairwise accuracy on the held-out target system is comparable to the baseline_single columns of Tables 3 and 4; counting-fingerprint MinMax results should match the maccsCount result path in results/PredRet/v2/final.
- Counting fingerprints are expected to give superior pairwise accuracy over binary fingerprints (a finding of the paper), so a MinMax/maccsCount run underperforming a Tanimoto/maccs run warrants rechecking fingerprint computation.

## Limitations

- The full pairwise kernel matrix is never materialized — Q must be decomposed as A K_m A^T; attempting to build the full pairwise matrix for large preference sets P(s) is infeasible.
- MACCS counting fingerprints require the modified cdkr fork installed via rcdkTools (install_github with ref=support_cdk_substructure_fps), which overwrites any existing rcdk installation and is not supported by the original package.
- Retained molecules must be pre-filtered: keep the lowest retention time and drop molecules whose lowest and largest RT differ by more than 5%; also remove molecules with very small retention times that do not interact with the chromatographic system.
- Java JDK >= 1.8 is required by CDK; rJava class-version errors can occur with prepackaged CRAN rJava when multiple Java versions are installed.

## Evidence

- [methods] Build the molecular kernel matrix K_m with [K_m]_ij = phi(m_i)^T phi(m_j), using the Min Max kernel for counting fingerprints and the Tanimoto kernel for binary fingerprints.: [evidence span withheld — anchor status: not_found]
- [methods] Molecular structures are represented using MACCS fingerprints, which are calculated using the rcdk R-package given the InChI representation of the molecular structures.: [evidence span withheld — anchor status: not_found]
- [methods] decomposing Q = A K_m A^T to avoid materializing the full pairwise kernel matrix, and select the regularization parameter C via a parallel hyper-parameter grid search: [evidence span withheld — anchor status: not_found]
