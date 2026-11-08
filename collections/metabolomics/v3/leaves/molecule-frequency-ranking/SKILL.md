---
name: molecule-frequency-ranking
description: Use when a generative model (e.g. DiffMS's graph diffusion decoder) outputs
  many sampled molecular structures per mass spectrum and you need a deterministic
  ranked candidate list for top-k (k=1,10) elucidation evaluation.
when_to_use_negative:
- When ranking an MSNovelist baseline, whose candidates are instead produced by beam
  search with width 100 and ranked by computed log-likelihoods rather than generation
  frequency
- When only a single generated structure per spectrum exists (frequency counting is
  meaningless without repeated samples)
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: RDKit
  role: Validates each sampled molecular structure and flags invalid or disconnected
    molecules for removal before frequency counting (tested version 2024.09.4)
- name: DiffMS
  role: Produces the pool of 100 sampled candidate molecules per spectrum that this
    filter ranks
  repo: github.com/coleygroup/DiffMS
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:491360552e6f72c04ff9c795dc7072044c4cbfd1e3d361d42f63671bece765bc
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
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# molecule-frequency-ranking

## Summary

Post-processing filter for diffusion-based de novo structure generation: validate sampled molecules with RDKit, drop invalid/disconnected structures, rank distinct candidates by generation frequency per spectrum, and select the top-k to form the per-spectrum candidate pool.

## When to use

Use when a generative model (e.g. DiffMS's graph diffusion decoder) outputs many sampled molecular structures per mass spectrum and you need a deterministic ranked candidate list for top-k (k=1,10) elucidation evaluation.

## When NOT to use

- When ranking an MSNovelist baseline, whose candidates are instead produced by beam search with width 100 and ranked by computed log-likelihoods rather than generation frequency
- When only a single generated structure per spectrum exists (frequency counting is meaningless without repeated samples)

## Inputs

- Diffusion sampling output: 100 sampled molecules per spectrum (across 5 predicted formulae per spectrum)
- Per-spectrum identifiers (MassSpecGym or NPLIB1 spectra)

## Outputs

- ranked_candidates.csv with one frequency-ranked candidate list per spectrum
- Set of discarded invalid/disconnected sampled molecules

## How to apply

Load the diffusion sampling output containing 100 sampled molecules per spectrum, split across the 5 predicted formulae (e.g. from the MIST formula transformer). Validate each sampled structure with RDKit, discarding molecules that are invalid or disconnected. Count the generation frequency of each distinct valid molecule within each spectrum. Rank candidates by frequency and select the top-k (k=1 and k=10 as used in the paper's evaluation). Write the per-spectrum ranked candidate list to ranked_candidates.csv, applying this identical post-processing to DiffMS and all baseline methods so rankings are comparable.

## Related tools

- **RDKit** (Validates each sampled molecular structure and flags invalid or disconnected molecules for removal before frequency counting (tested version 2024.09.4))
- **DiffMS** (Produces the pool of 100 sampled candidate molecules per spectrum that this filter ranks) — github.com/coleygroup/DiffMS

## Examples

```
mol = Chem.MolFromSmiles(smi); valid = mol is not None and len(Chem.GetMolFrags(mol)) == 1  # keep only valid, connected molecules before frequency counting
```

## Evaluation signals

- ranked_candidates.csv contains exactly one ranked candidate list per spectrum
- Every molecule retained in the ranked list passes RDKit validity and connectivity checks (no fragments)
- Ranking order matches descending generation frequency of distinct molecules within each spectrum
- The same filtering and top-k (k=1,10) protocol was applied to both DiffMS and baseline methods
- Total valid candidates per spectrum never exceeds 100 minus discarded invalid/disconnected samples

## Limitations

- Frequency ranking depends on sampling multiplicity, so it is only meaningful with repeated sampling (100 per spectrum); the same protocol must be applied uniformly to baselines or comparisons become unfair
- Discrete diffusion can sample invalid or disconnected graphs; RDKit-based rejection reduces but does not guarantee chemical plausibility of the retained candidates

## Evidence

- [methods] sampling and ranking protocol: [evidence span withheld — anchor status: not_found]
- [other] candidate pool composition: [evidence span withheld — anchor status: not_found]
- [other] uniform post-processing across methods: [evidence span withheld — anchor status: not_found]
