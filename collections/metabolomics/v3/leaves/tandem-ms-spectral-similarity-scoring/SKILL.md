---
name: tandem-ms-spectral-similarity-scoring
description: Use when you have preprocessed MS/MS peak lists (m/z + intensity) and
  need a pairwise similarity matrix (e.g. cosine scores for MST edges in MMSA, or
  GNPS networking edges between spectra in a molecular family).
when_to_use_negative:
- Input spectra have already been pairwise-aligned or transitively aligned — this
  skill scores pairs only, it does not produce multi-spectrum consensus peak sets
  (use MMSA alignment instead).
- You need shifted-peak detection across spectra (e.g. C12 HAA mass shifts in rhamnolipids)
  — cosine scoring of unmodified peaks does not capture those shifts, as noted in
  the article's limitations.
- The goal is library spectral matching to reference databases rather than pairwise
  similarity between experimental spectra in a network.
edam_operation: http://edamontology.org/operation_0405
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MMSA
  role: Wraps the pairwise modified-cosine (_cosine_fast) scoring as the similarity
    layer of its multiple spectral alignment pipeline
- name: NetworkX 3.3
  role: Consumes the 1−similarity edge weights to build the minimum spanning tree
    over spectra within a molecular network component
- name: GNPS2
  role: Source of molecular network tasks (task ID + component number) from which
    MS/MS spectra to score are retrieved via USIs like mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan>
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:a74b135230c4d43779f24cd737883ffbd98f3745ccd9065f1cb34fd10ce497a2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  edam_operation: http://edamontology.org/operation_0405
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tandem-ms-spectral-similarity-scoring

## Summary

Scores MS/MS spectrum pairs using a modified cosine similarity (_cosine_fast) after top-N peak filtering, square-root intensity transformation, and normalization — the pairwise scoring layer that feeds multiple spectral alignment (MMSA) and molecular networking edge construction.

## When to use

Use when you have preprocessed MS/MS peak lists (m/z + intensity) and need a pairwise similarity matrix (e.g. cosine scores for MST edges in MMSA, or GNPS networking edges between spectra in a molecular family).

## When NOT to use

- Input spectra have already been pairwise-aligned or transitively aligned — this skill scores pairs only, it does not produce multi-spectrum consensus peak sets (use MMSA alignment instead).
- You need shifted-peak detection across spectra (e.g. C12 HAA mass shifts in rhamnolipids) — cosine scoring of unmodified peaks does not capture those shifts, as noted in the article's limitations.
- The goal is library spectral matching to reference databases rather than pairwise similarity between experimental spectra in a network.

## Inputs

- Preprocessed MS/MS spectrum peak lists (m/z and intensity pairs) per scan, e.g. from an MGF such as GNPS2 specs_ms.mgf
- Precursor ion m/z per spectrum (for precursor-peak exclusion)
- GNPS task ID or USI list identifying spectra to compare

## Outputs

- Pairwise spectrum similarity score matrix (modified cosine scores)
- Edge-weighted graph of spectra (weights = 1 − similarity) suitable for MST construction
- Per-pair peak matches (aligned peak pairs) from the cosine alignment

## How to apply

Preprocess each MS/MS spectrum by filtering to the top 6 most intense peaks within a ±25 m/z window, excluding peaks within ±17 m/z of the precursor ion, then apply a square-root intensity transformation and normalize intensities. Score every spectrum pair with the modified cosine algorithm (_cosine_fast) to produce a similarity matrix. For MMSA's MST-based alignment, convert each pairwise similarity score to an edge weight by subtracting it from 1 (lower weight = more similar), and build the minimum spanning tree over spectra with NetworkX 3.3. Rationale: the top-N/±25 m/z filter reduces noise peaks that dominate cosine scores, and precursor ±17 m/z exclusion removes precursor/adduct peaks that carry no fragment information; square-root transformation dampens outlier intensities so the cosine score reflects shared fragment structure rather than a few dominant peaks.

## Related tools

- **MMSA** (Wraps the pairwise modified-cosine (_cosine_fast) scoring as the similarity layer of its multiple spectral alignment pipeline)
- **NetworkX 3.3** (Consumes the 1−similarity edge weights to build the minimum spanning tree over spectra within a molecular network component)
- **GNPS2** (Source of molecular network tasks (task ID + component number) from which MS/MS spectra to score are retrieved via USIs like mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan>)

## Examples

```
python -c "import networkx as nx; G = nx.Graph(); G.add_weighted_edges_from([(a, b, 1 - sim[a][b]) for a, b in pairs]); mst = nx.minimum_spanning_tree(G); order = list(nx.topological_sort(mst))"
```

## Evaluation signals

- Verify the similarity matrix is symmetric with diagonal 1.0 (self-similarity) and all off-diagonal values in [0, 1].
- Confirm MST edge weights equal 1 − pairwise similarity for each corresponding spectrum pair (lower weight = higher cosine similarity).
- Check that peak filtering was applied: after preprocessing, each spectrum retains at most the top 6 peaks per ±25 m/z window and no peaks within ±17 m/z of the precursor m/z.
- Downstream JSON output must exist, parse, and contain the MST/alignment structure with peak sets annotated with m/z and intensity values per spectrum in the component.
- Re-run scoring on a duplicate spectrum pair and confirm the cosine score is reproducible (deterministic given identical peak lists).

## Limitations

- The greedy multiple alignment does not fully capture smaller peak alignments (e.g. y2 peaks in the albicidin family).
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7) — the pairwise cosine layer misses large mass shifts.
- No repository hosting URL (github.com/owner/repo) is provided in the article; only the local directory name NetworkFamily_MultipleAlignment_Website is shown, so tool acquisition must be resolved separately.
- MST construction relies on NetworkX 3.3's greedy spanning-tree approach — topology-sensitive; alternative MST implementations may yield different alignment orders.
- Peak-set interpretation requires chemical context: aligned sets may mix fragment types unless spectra are from a coherent molecular family (e.g. rhamnolipid family sharing m/z 135.017 rhamnose fragment).

## Evidence

- [results] Preprocess each MS/MS spectrum by filtering peaks (top 6 most intense peaks within a +/- 25 m/z window, excluding peaks within +/- 17 m/z of the precursor ion), applying square root transformation to intensities, and normalizing intensities.: [evidence span withheld — anchor status: not_found]
- [results] Edge weights derived from pairwise similarity scores.: [evidence span withheld — anchor status: not_found]
- [other] MMSA's spectral alignment is computed by building an MST over spectra with nodes representing spectra and edges weighted by pairwise modified cosine similarity scores.: [evidence span withheld — anchor status: not_found]
