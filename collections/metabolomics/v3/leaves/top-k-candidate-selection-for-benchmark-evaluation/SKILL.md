---
name: top-k-candidate-selection-for-benchmark-evaluation
description: Use when evaluating de novo structure generation models (e.g., DiffMS
  on NPLIB1/MassSpecGym) whose raw sampler emits many molecules per spectrum and a
  ranked top-1/top-10 candidate list is needed for benchmark scoring.
when_to_use_negative:
- The formula pool is not yet predicted — this filter assumes the per-spectrum candidate
  pool (100 molecules across 5 formulae) already exists; run formula inference first
  (e.g., MIST or SIRIUS).
- Evaluating a baseline whose ranking mechanism is intrinsic to the method, e.g.,
  MSNovelist, which uses beam search with width 100 and log-likelihood ranking instead
  of frequency-based top-k selection.
tools:
- name: RDKit
  role: Validates each sampled molecule structure and detects disconnected (multi-component)
    molecules prior to frequency ranking; installed via rdkit=2024.09.4 in the diffms
    conda environment
- name: DiffMS
  role: Source of the sampled candidate pool — the discrete graph diffusion sampler
    generates the 100 molecules per spectrum that this filter post-processes
  repo: github.com/coleygroup/DiffMS
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:b41519b7ea8e26eb8073e0d79ada9b4be2de43c182f448b6d9d5f7510b1caeb4
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DiffMS
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Top-k candidate selection for benchmark evaluation

## Summary

Post-processing filter that converts DiffMS's raw diffusion sampling output (100 sampled molecules per spectrum across 5 predicted formulae) into frequency-ranked top-k candidate lists (k=1,10) by removing invalid or disconnected molecules with RDKit, enabling apples-to-apples benchmark comparison against baselines.

## When to use

Use when evaluating de novo structure generation models (e.g., DiffMS on NPLIB1/MassSpecGym) whose raw sampler emits many molecules per spectrum and a ranked top-1/top-10 candidate list is needed for benchmark scoring.

## When NOT to use

- The formula pool is not yet predicted — this filter assumes the per-spectrum candidate pool (100 molecules across 5 formulae) already exists; run formula inference first (e.g., MIST or SIRIUS).
- Evaluating a baseline whose ranking mechanism is intrinsic to the method, e.g., MSNovelist, which uses beam search with width 100 and log-likelihood ranking instead of frequency-based top-k selection.

## Inputs

- DiffMS diffusion sampling output: 100 sampled molecules (SMILES) per spectrum, distributed across the 5 predicted formulae
- RDKit-parseable SMILES strings for each sampled structure
- top-k size setting (k=1, k=10 as used in the paper's evaluation)

## Outputs

- ranked_candidates.csv containing one frequency-ranked candidate list per spectrum
- Per-spectrum distinct-molecule frequency counts after invalid/disconnected removal

## How to apply

Load the sampled molecule set (100 molecules per spectrum, split across the 5 predicted formulae) from the DiffMS diffusion sampling output. Parse each sampled SMILES with RDKit and discard molecules that are invalid or disconnected (multi-component), since the paper applies this removal uniformly to DiffMS and all baseline methods. Count the generation frequency of each distinct valid molecule per spectrum — frequency acts as an implicit confidence signal from the diffusion sampler. Rank candidates by descending frequency and truncate to top-k, using the paper's evaluation settings k=1 and k=10. Write one ranked candidate list per spectrum to ranked_candidates.csv, which downstream evaluation consumes identically for DiffMS and baselines so that any performance difference is attributable to the generative model rather than the post-processing.

## Related tools

- **RDKit** (Validates each sampled molecule structure and detects disconnected (multi-component) molecules prior to frequency ranking; installed via rdkit=2024.09.4 in the diffms conda environment)
- **DiffMS** (Source of the sampled candidate pool — the discrete graph diffusion sampler generates the 100 molecules per spectrum that this filter post-processes) — github.com/coleygroup/DiffMS

## Examples

```
from rdkit import Chem; valid = [s for s in sampled_smiles if Chem.MolFromSmiles(s) is not None and len(Chem.GetMolFrags(Chem.MolFromSmiles(s)))==1]
```

## Evaluation signals

- ranked_candidates.csv contains exactly one ranked candidate list per spectrum in the evaluated dataset (NPLIB1 or MassSpecGym split).
- Every SMILES surviving the filter parses in RDKit and represents a single connected component (invalid or disconnected molecules were removed).
- The candidate pool per spectrum does not exceed 100 samples distributed across the 5 predicted formulae before filtering.
- Top-k lists have length ≤ k for the evaluation settings k=1 and k=10, ordered by descending generation frequency of distinct valid molecules.
- The identical post-processing pipeline is applied to DiffMS and baseline outputs, so benchmark metrics are computed over the same filtering/ranking procedure.

## Limitations

- Frequency-based ranking is a heuristic; a molecule sampled more often is not guaranteed to be structurally closer to the true structure, since de novo generation of the exact molecular structure remains challenging.
- Discarding invalid or disconnected samples reduces the effective candidate pool below 100 for spectra where the diffusion sampler frequently emits degenerate graphs.
- The same filtering and ranking must be applied uniformly to DiffMS and baseline methods; applying it selectively would confound benchmark comparison.

## Evidence

- [methods] Sampling and ranking procedure: [evidence span withheld — anchor status: not_found]
