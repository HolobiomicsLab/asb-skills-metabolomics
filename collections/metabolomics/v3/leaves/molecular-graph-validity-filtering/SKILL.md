---
name: molecular-graph-validity-filtering
description: Use when a generative model (e.g., DiffMS discrete graph diffusion) has
  produced a pool of ~100 sampled molecules per mass spectrum and you need chemically
  valid, connected, frequency-ranked candidate structures for top-k evaluation (k=1,
  10).
when_to_use_negative:
- When candidate ranking should be model-score-based rather than frequency-based —
  e.g., the MSNovelist baseline in this paper uses beam search with width 100 and
  log-likelihood ranking instead of sampling frequency.
- When the true chemical formula is already given and a single formula-constrained
  generation is evaluated directly, rather than a pool assembled across 5 predicted
  formulae.
- When inputs are raw tandem mass spectra rather than already-sampled molecular structures
  — this skill operates downstream of spectrum-to-structure sampling.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: RDKit
  role: Validates each sampled molecule (parse structure, reject invalid or disconnected/multi-fragment
    molecules) before frequency counting
  repo: https://github.com/coleygroup/DiffMS
- name: DiffMS
  role: Source of the 100 sampled molecules per spectrum via discrete diffusion on
    the molecular adjacency matrix; the codebase adapted from DiGress contains the
    sampling and post-processing pipeline
  repo: https://github.com/coleygroup/DiffMS
- name: PyTorch
  role: Runtime for the diffusion sampling step that produces the candidate pool prior
    to filtering and ranking
  repo: https://github.com/coleygroup/DiffMS
- name: torch_geometric
  role: Graph representation library used by the DiGress-derived DiffMS codebase to
    handle sampled molecular graphs
  repo: https://github.com/coleygroup/DiffMS
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:99a4138787d62ba28dbcd7e158ac1fd8fd1bc70a5a900111afd6bd4925539fda
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
  - torch_geometric
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Molecular graph validity filtering and frequency-based candidate ranking

## Summary

Post-processing step for diffusion-based de novo molecule generation from mass spectra: validate each sampled molecular graph with RDKit, discard invalid or disconnected structures, then rank the remaining distinct molecules per spectrum by generation frequency and keep the top-k. This converts the raw diffusion sampler output (100 molecules per spectrum) into the ranked candidate lists used for elucidation evaluation.

## When to use

Use when a generative model (e.g., DiffMS discrete graph diffusion) has produced a pool of ~100 sampled molecules per mass spectrum and you need chemically valid, connected, frequency-ranked candidate structures for top-k evaluation (k=1, 10).

## When NOT to use

- When candidate ranking should be model-score-based rather than frequency-based — e.g., the MSNovelist baseline in this paper uses beam search with width 100 and log-likelihood ranking instead of sampling frequency.
- When the true chemical formula is already given and a single formula-constrained generation is evaluated directly, rather than a pool assembled across 5 predicted formulae.
- When inputs are raw tandem mass spectra rather than already-sampled molecular structures — this skill operates downstream of spectrum-to-structure sampling.

## Inputs

- DiffMS diffusion sampler output: 100 sampled molecules per mass spectrum (as SMILES or molecular graphs), split across the 5 predicted formulae
- Spectrum identifiers matching the evaluation set (MassSpecGym or NPLIB1/CANOPUS)

## Outputs

- ranked_candidates.csv containing one frequency-ranked candidate list per spectrum (top-k, k=1 and k=10)
- Per-spectrum counts of discarded invalid/disconnected molecules

## How to apply

Load the sampled molecule set produced by DiffMS's diffusion sampling — 100 molecules per spectrum, split across the 5 predicted formulae when the true formula is unknown. For each sampled structure, validate it with RDKit (2024.09.4 in the DiffMS environment): parse the molecule and reject it if it is invalid or disconnected (multiple fragments). Count how many times each distinct valid molecule was generated within the same spectrum's sample pool, since repeated sampling of the same structure is the signal the ranking exploits. Sort the distinct valid candidates by descending generation frequency and select the top-k (k=1 and k=10 in the paper's evaluation). Write one ranked candidate list per spectrum to ranked_candidates.csv, applying this exact post-processing uniformly to DiffMS and to baseline methods so rankings are comparable.

## Related tools

- **RDKit** (Validates each sampled molecule (parse structure, reject invalid or disconnected/multi-fragment molecules) before frequency counting) — https://github.com/coleygroup/DiffMS
- **DiffMS** (Source of the 100 sampled molecules per spectrum via discrete diffusion on the molecular adjacency matrix; the codebase adapted from DiGress contains the sampling and post-processing pipeline) — https://github.com/coleygroup/DiffMS
- **PyTorch** (Runtime for the diffusion sampling step that produces the candidate pool prior to filtering and ranking) — https://github.com/coleygroup/DiffMS
- **torch_geometric** (Graph representation library used by the DiGress-derived DiffMS codebase to handle sampled molecular graphs) — https://github.com/coleygroup/DiffMS

## Examples

```
from rdkit import Chem; mols = [Chem.MolFromSmiles(s) for s in sampled_smiles]; valid = [m for m in mols if m is not None and len(Chem.GetMolFrags(m)) == 1]
```

## Evaluation signals

- ranked_candidates.csv contains exactly one ranked candidate list per spectrum in the evaluation set, with no missing spectra.
- No candidate in the ranked lists is invalid or disconnected: every SMILES parses with RDKit MolFromSmiles and yields exactly one fragment via Chem.GetMolFrags.
- Each spectrum's pre-filter pool contains at most 100 samples (fewer after invalid/disconnected removal), drawn from the 5 predicted formulae.
- The ranked list length at k=1 and k=10 matches the paper's evaluation configuration, and ordering is strictly non-increasing in generation frequency.
- The identical filtering and top-k ranking is applied to DiffMS and all baseline methods, so cross-method comparisons use the same post-processing.

## Limitations

- Frequency-based ranking assumes the diffusion sampler repeatedly generates the correct structure; if sampling is highly diverse, the top-k by frequency may miss rare correct candidates.
- Removing invalid or disconnected molecules shrinks the effective candidate pool below the nominal 100 samples per spectrum, especially when formula constraints force many degenerate samples.
- The paper notes that exact molecular structure from de novo generation remains challenging — structurally close matches, not exact hits, may dominate even after correct filtering.

## Evidence

- [methods] Sampling 100 molecules per spectrum and ranking by frequency: [evidence span withheld — anchor status: not_found]
- [results] DiffMS generates 100 candidates split across predicted formulae: [evidence span withheld — anchor status: not_found]
- [methods] Uniform post-processing across methods: [evidence span withheld — anchor status: not_found]
