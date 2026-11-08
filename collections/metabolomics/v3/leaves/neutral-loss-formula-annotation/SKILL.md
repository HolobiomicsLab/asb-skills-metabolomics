---
name: neutral-loss-formula-annotation
description: Use when MS/MS spectra have been converted to fragmentation graphs of
  m/z differences and pattern edges need chemical interpretation, e.g. annotating
  a valine-like loss in GNPS network components.
when_to_use_negative:
- The m/z difference lies outside the 14–200 Da window over which neutral formulas
  are generated — no formula annotation is possible.
- You need probabilistic fragment/loss topic models rather than exact graph patterns;
  MS2LDA motifs (e.g. from MotifDB) are the complementary approach, and mineMS2 patterns
  are explicitly designed as complementary to them.
- The spectra have not been reduced to their most intense peaks (mineMS2 uses the
  15 most intense fragment ions per spectrum), so no reliable binned m/z-difference
  graph can be built.
tools:
- name: mineMS2
  role: Builds fragmentation graphs of m/z differences, mines exact patterns, computes
    closest molecular formulas for each m/z difference (14–200 Da), and couples patterns
    to GNPS components via infoPatterns/plotPatterns.
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Runtime environment; mineMS2 is delivered as an R package wrapped with Rcpp
    (C++/Boost back end).
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Provides the molecular network whose connected components (e.g. the 21-spectra
    fungisporin cluster) the formula-annotated patterns are evaluated against.
- name: igraph
  role: Used in R to load the GNPS network and identify the largest connected component
    of spectra for pattern-explanation scoring.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:c40f7137becee6ec5edc590bfe866576dc2e4e67df5e0e671db586c64c3ead0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# neutral-loss-formula-annotation

## Summary

Annotate fragmentation-graph edges (m/z differences) mined by mineMS2 with candidate molecular formulas, enabling chemical interpretation of losses such as the 99.0683 Da difference assigned to C5H9ON (valine loss) in the Penicillium nordicum fungisporin cluster.

## When to use

Use when MS/MS spectra have been converted to fragmentation graphs of m/z differences and pattern edges need chemical interpretation, e.g. annotating a valine-like loss in GNPS network components.

## When NOT to use

- The m/z difference lies outside the 14–200 Da window over which neutral formulas are generated — no formula annotation is possible.
- You need probabilistic fragment/loss topic models rather than exact graph patterns; MS2LDA motifs (e.g. from MotifDB) are the complementary approach, and mineMS2 patterns are explicitly designed as complementary to them.
- The spectra have not been reduced to their most intense peaks (mineMS2 uses the 15 most intense fragment ions per spectrum), so no reliable binned m/z-difference graph can be built.

## Inputs

- MS/MS peak lists / .mgf spectral library files (e.g. the 51-spectrum Penicillium nordicum dataset packaged as pnordicum.mgf)
- Spectral metadata table (e.g. dataset/pnordicum_ms2_info.tsv) passed to ms2Lib via suppInfos
- GNPS molecular network edges/components (e.g. a 21-spectra connected component) from igraph
- mineMS2-mined fragmentation patterns (graphs of m/z differences)

## Outputs

- Annotated pattern graphs in which each m/z-difference edge (14–200 Da) carries its closest candidate neutral molecular formula (e.g. 99.0683 → C5H9ON)
- Pattern/component explanation statistics: support (e.g. 19/21 spectra), precision (e.g. 1), and dataset-wide occurrence check
- Pattern structure plots over component spectra (plotPatterns) and a summary file of component statistics, edge annotation, support and precision

## How to apply

Represent each MS/MS spectrum as a directed acyclic fragmentation graph whose vertices are fragment ions and whose edges are binned m/z differences; bins are derived from the 15 most intense fragment ions per spectrum at a 15 ppm tolerance, chosen because lower values produced patterns too sensitive to small m/z variations. For interpretation, generate possible neutral formulas for all m/z differences between 14 and 200 Da (differences with no possible formula are discarded), so each pattern edge can be labeled with its closest molecular formula. When coupling to a GNPS molecular network, use infoPatterns to compute support and precision of candidate patterns over a component's spectra and select the pattern with highest coverage/precision — e.g. the 21-spectra P. nordicum fungisporin component is best explained by an edge at m/z difference 99.0683 with closest formula C5H9ON (valine loss), present in 19 of 21 spectra with precision 1 and no occurrences elsewhere in the dataset. Visualize and verify the mapping of the pattern onto component spectra with plotPatterns before saving annotation statistics.

## Related tools

- **mineMS2** (Builds fragmentation graphs of m/z differences, mines exact patterns, computes closest molecular formulas for each m/z difference (14–200 Da), and couples patterns to GNPS components via infoPatterns/plotPatterns.) — https://github.com/odisce/mineMS2
- **R** (Runtime environment; mineMS2 is delivered as an R package wrapped with Rcpp (C++/Boost back end).) — https://github.com/odisce/mineMS2
- **GNPS** (Provides the molecular network whose connected components (e.g. the 21-spectra fungisporin cluster) the formula-annotated patterns are evaluated against.)
- **igraph** (Used in R to load the GNPS network and identify the largest connected component of spectra for pattern-explanation scoring.)

## Examples

```
pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"); pnordicum_m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)
```

## Evaluation signals

- The annotated m/z difference of 99.0683 must be labeled with closest formula C5H9ON, consistent with a valine loss.
- The best-explaining pattern must have precision = 1 over the target component and no occurrences among other dataset spectra.
- Support must match the expected coverage: 19 of the 21 spectra of the GNPS fungisporin component.
- Exactness invariant: every m/z difference of a mineMS2 pattern is present in all spectra containing that pattern (exact graph structure).
- All formula-annotated edges must fall within the 14–200 Da neutral-formula generation window and the 15 ppm binning tolerance.

## Limitations

- m/z differences outside 14–200 Da cannot receive a candidate molecular formula.
- Exact pattern mining extracts a high number of patterns, and no scoring system for prioritizing the most informative patterns yet exists.
- Peak selection (based solely on peak intensities) is required due to combinatorial complexity and may lose chemically relevant information; formula-aware alternatives are not yet implemented.
- The DAG representation does not require knowledge of the precursor molecular formula, but incorporating this option is left to future versions of the package.

## Evidence

- [methods] Formula generation: Possible neutral formulas are generated for all the m/z differences between 14 and 200.: "Formula generation:  Possible neutral formulas are generated for all the m/z differences between 14 and 200."
- [results] 15 ppm tolerance rationale: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [methods] m/z difference binning from 15 most intense fragment ions: "A set of disjoint bins is derived from the full set of m/z differences between the 15 most intense fragment ions from each spectrum"
- [readme] Candidate molecular formulas computed for each m/z difference below 200 Da: "The candidate **molecular formulas for each m/z difference** are computed to help interpretation (below 200 Da)."
- [supplementary] Packaged P. nordicum dataset loading into ms2Lib: "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")"
