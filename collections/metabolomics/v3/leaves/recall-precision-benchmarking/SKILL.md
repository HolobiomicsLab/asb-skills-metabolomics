---
name: recall-precision-benchmarking
description: Use when a spectral dereplication tool must be quantitatively evaluated
  against spectra of known compounds, e.g. computing rank-1 recall/precision at a
  stated FDR cutoff like 3%.
when_to_use_negative:
- Spectra lack known-structure annotations, so recall against ground-truth compound
  identities cannot be computed
- Evaluating tools for compound classes DEREPLICATOR+ does not cover, or genome-mining/metabologenomic
  tasks (e.g. MetaMiner BGC analysis) rather than spectral benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DEREPLICATOR+
  role: Tool under evaluation; database search of each spectrum against the reference
    compound database, using decoy fragmentation graphs to estimate FDR
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Predecessor dereplication tool for peptidic natural products included in the
    same benchmarking framework
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:10ace6ab6ab14e0b9403b67a4bbf933c03026f7cbffe5ceba87718e56b4624e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# recall-precision-benchmarking

## Summary

Benchmarking protocol for evaluating a metabolite spectrum matching (MSM) tool such as DEREPLICATOR+ against a reference spectral library (SpectraLibrary), computing rank-1 and top-3 recall and precision at a fixed FDR threshold.

## When to use

Use when a spectral dereplication tool must be quantitatively evaluated against spectra of known compounds, e.g. computing rank-1 recall/precision at a stated FDR cutoff like 3%.

## When NOT to use

- Spectra lack known-structure annotations, so recall against ground-truth compound identities cannot be computed
- Evaluating tools for compound classes DEREPLICATOR+ does not cover, or genome-mining/metabologenomic tasks (e.g. MetaMiner BGC analysis) rather than spectral benchmarking

## Inputs

- SpectraLibrary spectral dataset: compounds with known structures and reference tandem mass spectra (5473 spectra)
- Reference compound database of natural product chemical structures
- DEREPLICATOR+ MSM output with target matches, decoy matches, and per-match scores (number of unique matched masses)

## Outputs

- Rank-1 recall (identified spectra / total spectra) and rank-1 precision (correct rank-1 / total rank-1 matches)
- Top-3 recall and top-3 precision
- Set of significant metabolite-spectrum matches retained at the chosen FDR threshold (e.g. 3087 identifications at 3% FDR)

## How to apply

Load the SpectraLibrary dataset of compounds with known structures and their reference tandem mass spectra, then run the tool (DEREPLICATOR+) to search each spectrum against the reference compound database, generating target matches and matching decoy fragmentation graphs to estimate the false discovery rate. Apply the FDR threshold (here 3%) to retain only significant metabolite-spectrum matches (MSMs), scored by the number of unique masses annotating nodes of the metabolite fragmentation graph. From the retained matches, compute rank-1 recall (identified spectra / total library spectra) and rank-1 precision (correct rank-1 matches / total rank-1 matches) by comparing identifications to the known compound annotations; repeat with correct compounds counted anywhere in the top-3 candidates. Validate against the published reference values: rank-1 recall 33% (1802/5473), rank-1 precision 58% (1802/3087), top-3 recall 48% (2614/5473), top-3 precision 85% (2614/3087).

## Related tools

- **DEREPLICATOR+** (Tool under evaluation; database search of each spectrum against the reference compound database, using decoy fragmentation graphs to estimate FDR) — https://github.com/ablab/npdtools
- **DEREPLICATOR** (Predecessor dereplication tool for peptidic natural products included in the same benchmarking framework) — https://github.com/ablab/npdtools

## Examples

```
./bin/dereplicator+.py --spectra spectra_library.mgf --database structures.sdf --fdr 0.03
```

## Evaluation signals

- Rank-1 recall equals 33% (1802 of 5473 SpectraLibrary spectra)
- Rank-1 precision equals 58% (1802 correct of 3087 identifications)
- Top-3 recall equals 48% (2614/5473) and top-3 precision equals 85% (2614/3087)
- MSM count before precision calculation matches the number of matches retained at the 3% FDR threshold

## Limitations

- DEREPLICATOR+ simple shared-peaks scoring results in a high false-negative rate, capping achievable recall
- DEREPLICATOR+ is not applicable to all classes of compounds
- Metrics depend on the FDR threshold and rank-1 definition; alternative thresholds or rank windows change the numbers
- P-value recursion is only approximate for non-tree fragmentation graphs, affecting FDR estimation

## Evidence

- [results] On the SpectraLibrary benchmark at 3% FDR, defining success as a rank-1 correct prediction, DEREPLICATOR+ achieves a recall rate of 33% (1802 out of 5473 spectra) and a precision rate of 58% (1802 out of 3087 identifications).: [evidence span withheld — anchor status: not_found]
- [discussion] scoring by unique masses: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M"
- [discussion] decoy-based FDR estimation: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [readme] DEREPLICATOR+ availability: [evidence span withheld — anchor status: not_found]
