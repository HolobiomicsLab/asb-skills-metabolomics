---
name: cut-vertex-fragmentation-graph-construction
description: Use when scoring a query mass spectrum (e.g. mzML/MGF from MSV000086287)
  against many candidate molecules and you need a metabolite graph whose fragmentation-graph
  nodes are computed on demand rather than exhaustively.
when_to_use_negative:
- The query spectrum is already preprocessed — the README warns peak merging/filtering
  advanced settings 'should be left unchanged unless you have already done some preprocessing
  on your input spectra before uploading'
- You need an exhaustive precomputed fragment library rather than an adaptive, on-demand
  expansion driven by query peaks
- Precursor masses do not match any candidate in exact mode, or the variant-interpretation
  Δ logic of variable mode is the goal rather than depth-1 cut-vertex fragmentation
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: VInSMoC
  role: Implements the metabolite graph construction, Hopcroft–Tarjan cut-vertex finding,
    and adaptive fragmentation-graph expansion that this skill describes
  repo: https://github.com/mohimanilab/VInSMoC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:84ec6ec1cfdb7b8f8ce077225635bcd2b8df824eb0026c91692fddcc8efaf945
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cut-vertex-fragmentation-graph-construction

## Summary

Builds a fragmentation graph over each candidate molecule's bonds and atoms, anchoring nodes on Hopcroft–Tarjan cut vertices so that only depth-1 fragment nodes need to be materialized up front and deeper nodes are expanded lazily during scoring.

## When to use

Use when scoring a query mass spectrum (e.g. mzML/MGF from MSV000086287) against many candidate molecules and you need a metabolite graph whose fragmentation-graph nodes are computed on demand rather than exhaustively.

## When NOT to use

- The query spectrum is already preprocessed — the README warns peak merging/filtering advanced settings 'should be left unchanged unless you have already done some preprocessing on your input spectra before uploading'
- You need an exhaustive precomputed fragment library rather than an adaptive, on-demand expansion driven by query peaks
- Precursor masses do not match any candidate in exact mode, or the variant-interpretation Δ logic of variable mode is the goal rather than depth-1 cut-vertex fragmentation

## Inputs

- Candidate molecule (from NPAtlas chemical database) parsed into a metabolite graph
- Preprocessed query mass spectrum (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287, positive ion mode)

## Outputs

- Fragmentation graph of depth-1 fragment nodes per candidate molecule, with lazily expanded higher-depth nodes
- Set of explained parent fragments used to guide further graph expansion during scoring
- Theoretical fragment masses matched to experimental peaks (within product ion tolerance) feeding the peak count score

## How to apply

First build the metabolite graph for each candidate molecule from the chemical database (e.g. NPAtlas) via a single-pass depth-first traversal, then identify articulation points with Hopcroft–Tarjan cut-vertex finding to construct depth-1 fragmentation graph nodes. Compute a deeper fragmentation-graph fragment only when its parent fragment has been explained by a query peak; fragments computed at depth greater than 1 are stored to avoid duplicate computation. Convert query peak m/z values to neutral masses with the [M+H]+ adduct (positive ion mode, maximum absolute charge 1), and match theoretical fragments to experimental peaks within the product ion tolerance before computing the exact-mode peak count score. Preprocess the spectrum first: merge peaks within the Peak Merge Threshold while retaining at least Minimum Peak Count peaks, then keep only the Peaks Per Window most intense peaks in m/z windows of Peak Filtering Window.

## Related tools

- **VInSMoC** (Implements the metabolite graph construction, Hopcroft–Tarjan cut-vertex finding, and adaptive fragmentation-graph expansion that this skill describes) — https://github.com/mohimanilab/VInSMoC

## Examples

```
vinSMoC --spectrum extract-3_Metal-mix_1mM.mzML --database NPAtlas --search-mode exact --fragmentation-mode Metabolite --ion-mode positive --max-charge 1 --min-score 0 --output vinSMoC_exact_mode_hits.csv
```

## Evaluation signals

- Depth > 1 fragmentation-graph fragments exist only for parents whose fragments were explained by a query peak
- No fragment is recomputed: every computed higher-depth fragment is stored and reused
- All fragment-to-peak matches fall within the configured product ion tolerance
- Reported hits have peak count scores exceeding the minimum score reporting threshold
- Each reported molecule links back to its original chemical database entry

## Limitations

- Deeper fragments are never computed unless a parent fragment is explained by a query peak, so the graph is incomplete by design until scoring drives expansion
- Adduct conversion is limited by maximum absolute charge; with charge 1 only [M+H]+ or [M-H]- adducts are used based on ion mode
- Fragmentation mode must match the molecule class (Metabolite mode for generic molecules vs. Peptide mode for generalized peptides)

## Evidence

- [methods] Adaptive expansion rule: [evidence span withheld — anchor status: not_found]
- [other] Cut-vertex construction: [evidence span withheld — anchor status: not_found]
