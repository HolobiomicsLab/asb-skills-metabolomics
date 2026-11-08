---
name: rdkit-molecule-structure-validation
description: Use when a generative model (e.g. DiffMS discrete graph diffusion) has
  produced a pool of sampled molecules per spectrum and you must filter out invalid
  or disconnected structures before ranking or evaluation.
when_to_use_negative:
- Input candidates are already validated or come from curated databases (e.g. PubChem,
  HMDB library entries) rather than de novo generation sampling
- The task requires preserving multi-component mixtures or salts as legitimate candidates
  rather than single connected structures
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Parses and sanitizes each sampled molecule SMILES/graph and detects disconnected
    (multi-fragment) structures for removal; pinned to rdkit=2024.09.4 in the diffms
    conda environment
  repo: https://github.com/coleygroup/DiffMS
- name: DiffMS
  role: Source of the sampled molecule pool (100 molecules per spectrum via discrete
    graph diffusion) that the validation filter is applied to
  repo: https://github.com/coleygroup/DiffMS
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:dd2f63e6556f19ac4530b448f3f2d037312e517b7ef22b9560bc1d64a81198fe
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiffMS
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# RDKit molecule structure validation

## Summary

Validates sampled molecular structures with RDKit, discarding molecules that are invalid (unparseable/sanitization-failing SMILES) or disconnected (multiple fragments), so that downstream frequency-based candidate ranking operates only on chemically well-formed single-structure candidates from the DiffMS diffusion sampling output.

## When to use

Use when a generative model (e.g. DiffMS discrete graph diffusion) has produced a pool of sampled molecules per spectrum and you must filter out invalid or disconnected structures before ranking or evaluation.

## When NOT to use

- Input candidates are already validated or come from curated databases (e.g. PubChem, HMDB library entries) rather than de novo generation sampling
- The task requires preserving multi-component mixtures or salts as legitimate candidates rather than single connected structures

## Inputs

- Sampled molecule set per spectrum from DiffMS diffusion sampling output (100 molecules per spectrum, across 5 predicted formulae), as SMILES strings or molecular graphs
- Baseline-method candidate molecule lists requiring the same uniform post-processing

## Outputs

- Filtered set of valid, connected (single-fragment) molecule structures per spectrum
- Discard log / counts of invalid and disconnected molecules removed before frequency ranking

## How to apply

Load the per-spectrum sampled molecule set from the DiffMS diffusion sampling output — 100 molecules per spectrum, split across the 5 predicted formulae — as SMILES or molecular graphs. Parse each structure with RDKit (version 2024.09.4 in the paper's conda environment); molecules that fail parsing/sanitization are 'invalid' and are discarded outright. For parseable molecules, check connectedness with RDKit fragment splitting (Chem.GetMolFrags) and discard any molecule with more than one fragment ('disconnected'). Apply this uniform post-processing to both DiffMS and baseline methods so rankings are comparable, then pass the surviving valid, connected molecules to frequency counting and top-k (k=1, 10) selection written to ranked_candidates.csv.

## Related tools

- **RDKit** (Parses and sanitizes each sampled molecule SMILES/graph and detects disconnected (multi-fragment) structures for removal; pinned to rdkit=2024.09.4 in the diffms conda environment) — https://github.com/coleygroup/DiffMS
- **DiffMS** (Source of the sampled molecule pool (100 molecules per spectrum via discrete graph diffusion) that the validation filter is applied to) — https://github.com/coleygroup/DiffMS

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smi); valid = mol is not None and len(Chem.GetMolFrags(mol)) == 1
```

## Evaluation signals

- No molecule in the filtered set fails RDKit MolFromSmiles/sanitization
- Every retained molecule has exactly one RDKit fragment (GetMolFrags returns a single fragment)
- The filtered pool derives from exactly 100 sampled molecules per spectrum (minus discarded invalid/disconnected ones) across the 5 predicted formulae
- The same invalid/disconnected removal is applied identically to DiffMS and all baseline methods before top-k frequency ranking, so ranked_candidates.csv comparisons are apples-to-apples

## Limitations

- Validity and connectedness are RDKit-level checks; they do not guarantee chemical plausibility or correct formula assignment beyond the diffusion model's heavy-atom constraint
- Aggressive filtering can shrink the candidate pool, affecting subsequent frequency-based top-k (k=1, 10) ranking statistics
- DiGress-derived codebase carries a reported bug note (dim_ffy ignored in GraphTransformer, default 2048 used) whose impact on sampled structures is unknown

## Evidence

- [methods] Sampling and frequency ranking filter: [evidence span withheld — anchor status: not_found]
