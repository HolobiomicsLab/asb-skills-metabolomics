---
name: connectivity-matrix-topology-feature-engineering
description: Use when retraining or extending a competitive-fragmentation-model predictor
  on QTOF ESI-MS/MS training sets (e.g. Metlin [M+H]+/[M-H]- at 10/20/40 eV) and you
  need bond-topology-aware fragment features, or when running component-ablation to
  quantify topology-feature contribution.
when_to_use_negative:
- The task is EI-MS/MS prediction or compound identification scoring rather than ESI-MS/MS
  fragment feature modeling — topology features were evaluated on QTOF ESI-MS/MS prediction.
- Input spectra come from non-QTOF instruments, which CFM-ID 4.0 explicitly does not
  yet handle.
- You only need the hand-written rule-based pathway (MSRB classes such as acylcarnitines,
  acylcholines, flavonoids) — rule-based features contributed only ~1% and covered
  under 3% of spectra.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3315
tools:
- name: CFM-ID 4.0
  role: Implements the adjacency-matrix structure representation, topology feature
    extraction, and NN training/ablation over Metlin QTOF data
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Baseline competitive fragmentation model whose parameterization lacks topology
    features, used as ablation baseline
- name: CFM-ID 3.0
  role: Intermediate baseline adding hand-coded fragmentation rules but no connectivity-matrix
    topology features
- name: Metlin
  role: Source of experimental QTOF ESI-MS/MS spectra used to train and evaluate topology-feature
    variants
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:09a8ddd5938da73c031e024d03478d236325e4a9d4add2fbf0485f20b5a75093
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Metlin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# Connectivity-matrix topology feature engineering

## Summary

Represents each candidate fragment as an adjacency (connectivity) matrix from which MS-relevant topological features are extracted, so CFM-ID 4.0's neural predictor learns fragmentation parameters from chemical bond topology rather than atom-identity-only descriptors. Ablation attributes ~65% of CFM-ID 4.0's ESI-MS/MS gain on Metlin QTOF data to these topology features.

## When to use

Use when retraining or extending a competitive-fragmentation-model predictor on QTOF ESI-MS/MS training sets (e.g. Metlin [M+H]+/[M-H]- at 10/20/40 eV) and you need bond-topology-aware fragment features, or when running component-ablation to quantify topology-feature contribution.

## When NOT to use

- The task is EI-MS/MS prediction or compound identification scoring rather than ESI-MS/MS fragment feature modeling — topology features were evaluated on QTOF ESI-MS/MS prediction.
- Input spectra come from non-QTOF instruments, which CFM-ID 4.0 explicitly does not yet handle.
- You only need the hand-written rule-based pathway (MSRB classes such as acylcarnitines, acylcholines, flavonoids) — rule-based features contributed only ~1% and covered under 3% of spectra.

## Inputs

- Metlin 2015/2019 QTOF ESI-MS/MS training/test sets ([M+H]+ and [M-H]- at 10/20/40 eV collision energies)
- Experimental peak lists for Dice/Dot Product scoring
- Candidate chemical structures (SMILES/Molfile) decomposed into fragmentation graphs (depth 2, max ring breaks 2)

## Outputs

- Adjacency-matrix-based connectivity/topological feature vectors per fragment transition
- Trained CFM-ID 4.0 topology-feature model variant
- Component contribution shares table (topology ~65%, ring cleavage ~35%, rule-based ~1%)

## How to apply

Convert each molecule and its combinatorially generated fragments (fragmentation graph depth 2, max 2 ring breaks, ring cleavage modeled as sequential simple bond dissociations) into an adjacency-matrix representation, then extract MS-relevant topological features such as bond types and connectivity contexts of the reacting bond. Encode these features as inputs to the fully connected NN (two 128-node hidden layers) trained under the 10-fold cross-validation framework on Metlin 2015/2019 QTOF ESI-MS/MS sets. The rationale, per the authors, is that encoding more information about chemical bond energies/topology improves ESI-MS/MS fragmentation modeling over CFM-ID 2.0/3.0 descriptors. To quantify contribution, train ablation variants — baseline without connectivity-matrix topology features, then adding them, then adding sequential ring cleavage, then adding MSRB rules — and score each with Dice coefficient and re-weighted Stein Dot Product at a peak-match tolerance of m/z < 0.01 Da or 10 ppm (whichever first), attributing the delta between full CFM-ID 4.0 and baseline to each component (reported: ~65% topology, 35% ring cleavage, ~1% rule-based).

## Related tools

- **CFM-ID 4.0** (Implements the adjacency-matrix structure representation, topology feature extraction, and NN training/ablation over Metlin QTOF data) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Baseline competitive fragmentation model whose parameterization lacks topology features, used as ablation baseline)
- **CFM-ID 3.0** (Intermediate baseline adding hand-coded fragmentation rules but no connectivity-matrix topology features)
- **Metlin** (Source of experimental QTOF ESI-MS/MS spectra used to train and evaluate topology-feature variants)

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Topology-feature ablation variant loses the largest share of full-model performance; reproduced contribution share for molecular topology features lands within a few percentage points of the reported ~65%.
- Ring-cleavage share reproduces near the reported ~35% and rule-based near ~1%, with MSRB predicting fewer than 3% of all MS/MS spectra.
- Dice coefficient and re-weighted Stein Dot Product improve monotonically as topology features are added to the baseline.
- Peak matching respects the stated tolerance: m/z difference < 0.01 Da or 10 ppm, whichever comes first.
- Training uses the stated setup: 10-fold cross-validation, fragmentation graph depth 2, max ring breaks 2, NN with two 128-node hidden layers.

## Limitations

- Deeper fragmentation graphs, which yield richer connectivity features, introduce extra computational complexity.
- Runtime limitations are acknowledged without exact benchmarks; accuracy/runtime improvements remain open.
- Feature gains were demonstrated on QTOF ESI-MS/MS data; handling of other instrument types is planned future work.

## Evidence

- [discussion] Deeper fragmentation graphs add computational cost: [evidence span withheld — anchor status: not_found]
