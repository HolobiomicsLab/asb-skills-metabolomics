---
name: tanimoto-mces-metric-computation
description: Use when you have ranked candidate molecular structures sampled per spectrum
  (e.g., 100 DiffMS diffusion samples with invalid/disconnected molecules removed)
  and need to score them against ground-truth molecules for benchmark reproduction.
when_to_use_negative:
- When evaluating fingerprint prediction itself (e.g., MIST encoder fingerprint accuracy)
  rather than generated molecular structures
- When candidates have not been filtered for validity/connectivity — scoring invalid
  structures against the MassSpecGym metrics is undefined and will distort rankings
- When the task is spectrum-to-fingerprint pretraining evaluation on DSSTox/HMDB/COCONUT/MOSES
  pairs rather than de novo generation benchmarking
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
tools:
- name: RDKit
  role: Morgan fingerprint computation for Tanimoto similarity, plus validity and
    connectivity checks on sampled molecules
  repo: github.com/coleygroup/DiffMS
- name: DiffMS
  role: Graph diffusion decoder that samples the 100 candidate molecules per spectrum
    which are then scored
  repo: github.com/coleygroup/DiffMS
- name: MassSpecGym
  role: Source of the de novo generation metric definitions (top-k accuracy, Tanimoto,
    MCES) used for scoring
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:3cac26fda2e589bf1a4db1530b409c0d5a15967ab1d2c9c145625e11d440fd4d
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DiffMS
  - MIST
  - MassSpecGym
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tanimoto-mces-metric-computation

## Summary

Evaluate de novo molecule generation candidates against ground-truth structures using top-k accuracy, top-k maximum Tanimoto similarity (RDKit Morgan fingerprints), and top-k minimum MCES distance, as defined for MassSpecGym-style de novo generation benchmarks such as NPLIB1.

## When to use

Use when you have ranked candidate molecular structures sampled per spectrum (e.g., 100 DiffMS diffusion samples with invalid/disconnected molecules removed) and need to score them against ground-truth molecules for benchmark reproduction.

## When NOT to use

- When evaluating fingerprint prediction itself (e.g., MIST encoder fingerprint accuracy) rather than generated molecular structures
- When candidates have not been filtered for validity/connectivity — scoring invalid structures against the MassSpecGym metrics is undefined and will distort rankings
- When the task is spectrum-to-fingerprint pretraining evaluation on DSSTox/HMDB/COCONUT/MOSES pairs rather than de novo generation benchmarking

## Inputs

- Ranked candidate molecular structure lists (top-1 and top-10) per test spectrum, after removal of invalid or disconnected molecules
- Ground-truth molecular structures for the NPLIB1 and MassSpecGym test sets
- RDKit Morgan fingerprint representations of candidate and ground-truth molecules

## Outputs

- Per-dataset top-k accuracy values (k=1, 10)
- Per-dataset top-k maximum Tanimoto similarity values
- Per-dataset top-k minimum MCES distance values (e.g., reproduction_results.csv)
- Comparison table against reported DiffMS Table 1 values

## How to apply

After sampling 100 molecules per test spectrum with the DiffMS graph diffusion decoder, use RDKit to check validity and connectivity, then rank candidates by frequency and take top-1 and top-10 lists. Score each ranked list against the ground-truth molecule with top-k accuracy, top-k maximum Tanimoto similarity computed from RDKit Morgan fingerprints, and top-k minimum MCES distance, following the MassSpecGym de novo generation metric definitions. Compile the resulting metrics per dataset (NPLIB1, MassSpecGym) into a results table and compare against the reported DiffMS Table 1 rows (e.g., NPLIB1 top-1 accuracy 8.34%, Tanimoto 0.35, MCES 11.95; MassSpecGym top-1 accuracy 2.30%, Tanimoto 0.28). Match within stochastic-sampling tolerance since diffusion decoding is nondeterministic.

## Related tools

- **RDKit** (Morgan fingerprint computation for Tanimoto similarity, plus validity and connectivity checks on sampled molecules) — github.com/coleygroup/DiffMS
- **DiffMS** (Graph diffusion decoder that samples the 100 candidate molecules per spectrum which are then scored) — github.com/coleygroup/DiffMS
- **MassSpecGym** (Source of the de novo generation metric definitions (top-k accuracy, Tanimoto, MCES) used for scoring)

## Examples

```
python3 -c "from rdkit import Chem; from rdkit.Chem import AllChem, DataStructs; fp=AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles('CCO'),2); print(fp.GetNumBits())"
```

## Evaluation signals

- Reproduced NPLIB1 top-1 accuracy should match the reported 8.34% (Tanimoto 0.35, MCES 11.95) within stochastic-sampling tolerance
- Reproduced MassSpecGym top-1 accuracy should match the reported 2.30% (Tanimoto 0.28) within tolerance
- All scored candidates pass RDKit validity and connectivity checks before metric computation
- Top-k lists are formed by frequency of sampled molecules across the 100 diffusion samples per spectrum
- Output metrics land in a defined results file (reproduction_results.csv) with file-existence and metric-value checks against the reference table

## Limitations

- Sampling-based evaluation is stochastic, so reproduced metrics only match reported values within tolerance, not exactly
- The top-k minimum MCES distance requires an MCES computation that can be expensive for structurally distant candidate/ground-truth pairs
- Metrics reflect candidate lists only after invalid or disconnected molecules are removed; without filtering, ranking by frequency changes

## Evidence

- [methods] Ranking and scoring protocol: [evidence span withheld — anchor status: not_found]
- [other] Metric definitions: [evidence span withheld — anchor status: not_found]
- [other] Reported reference values: [evidence span withheld — anchor status: not_found]
- [other] Stochastic tolerance: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark datasets: [evidence span withheld — anchor status: not_found]
