---
name: pattern-support-precision-evaluation
description: Use when a GNPS molecular network component of MS/MS spectra must be
  explained by one dominant fragmentation pattern, and candidate mineMS2 patterns
  need support/precision scoring over the component.
when_to_use_negative:
- When no molecular network component is defined and the goal is de novo pattern mining
  over a whole spectral library rather than explaining a specific component
- When the candidate patterns are MS2LDA motifs (distributions of fragments/losses
  rather than exact graphs), which cannot be verified as exact m/z-difference edges
  present in all containing spectra
- When m/z differences fall outside the 14–200 Da formula-generation window, since
  closest-formula annotation is not computed beyond 200 Da
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: mineMS2
  role: R package providing infoPatterns (support/precision computation over component
    spectra), plotPatterns (pattern visualization mapped onto spectra), and pattern-to-component
    coupling to GNPS networks
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Runtime environment; mineMS2 is an R package wrapped with Rcpp, executed as
    an R workflow
  repo: https://github.com/odisce/mineMS2
- name: igraph
  role: Used to load the GNPS molecular network and identify the largest connected
    component (21 spectra) whose spectra are scored
- name: GNPS
  role: Source of the molecular network whose components are the evaluation targets;
    each vertex is a spectrum and an edge indicates high pairwise similarity (modified
    cosine score)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:792286532433d5fd9e2270e44b6cc8f12d5614341a49ee293f6319dccf611994
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pattern-support-precision-evaluation

## Summary

Scores candidate mineMS2 fragmentation patterns against a GNPS molecular network component by computing support (number of component spectra containing the pattern) and precision (absence of the pattern elsewhere in the dataset), to identify the single pattern that best explains the component, e.g. the 21-spectra P. nordicum fungisporin cluster explained by a 99.0683 Da valine loss (C5H9ON).

## When to use

Use when a GNPS molecular network component of MS/MS spectra must be explained by one dominant fragmentation pattern, and candidate mineMS2 patterns need support/precision scoring over the component.

## When NOT to use

- When no molecular network component is defined and the goal is de novo pattern mining over a whole spectral library rather than explaining a specific component
- When the candidate patterns are MS2LDA motifs (distributions of fragments/losses rather than exact graphs), which cannot be verified as exact m/z-difference edges present in all containing spectra
- When m/z differences fall outside the 14–200 Da formula-generation window, since closest-formula annotation is not computed beyond 200 Da

## Inputs

- GNPS molecular network (component/edges) for the MS/MS dataset
- Spectral library object built with ms2Lib from an MGF file plus metadata TSV (e.g. pnordicum.mgf and pnordicum_ms2_info.tsv, 51 P. nordicum spectra)
- Set of mineMS2 fragmentation patterns previously mined from the dataset

## Outputs

- Per-pattern support and precision statistics over the component's spectra
- Annotated pattern edge: m/z difference with closest molecular formula (e.g. 99.0683 Da, C5H9ON, valine loss)
- plotPatterns figures showing the pattern mapped onto the component spectra
- Summary file with component statistics, pattern edge annotation, support, and precision

## How to apply

Build the spectral library with ms2Lib from the MGF plus a metadata TSV (e.g. pnordicum_ms2_info.tsv), then load the GNPS molecular network and the previously mined patterns into R. Identify the target connected component (e.g. 21 spectra) with igraph, and use the mineMS2 infoPatterns function to compute, for each pattern, support and precision over the component's spectra, selecting the pattern with highest coverage and precision. Verify the selected pattern contains the diagnostic edge — e.g. m/z difference 99.0683 whose closest generated formula (from the 14–200 Da formula-generation step) is C5H9ON, consistent with a valine loss — and that precision equals 1. Visualize the pattern mapped onto the component spectra with plotPatterns to confirm support (19 of 21 spectra), then check pattern occurrences across the full dataset to confirm no occurrence outside the component (precision = 1). Save component statistics, pattern edge annotation, support and precision into a summary file. Rationale: precision distinguishes a component-specific pattern from ubiquitous differences, which is what makes it an explanatory annotation for the network component.

## Related tools

- **mineMS2** (R package providing infoPatterns (support/precision computation over component spectra), plotPatterns (pattern visualization mapped onto spectra), and pattern-to-component coupling to GNPS networks) — https://github.com/odisce/mineMS2
- **R** (Runtime environment; mineMS2 is an R package wrapped with Rcpp, executed as an R workflow) — https://github.com/odisce/mineMS2
- **igraph** (Used to load the GNPS molecular network and identify the largest connected component (21 spectra) whose spectra are scored)
- **GNPS** (Source of the molecular network whose components are the evaluation targets; each vertex is a spectrum and an edge indicates high pairwise similarity (modified cosine score))

## Examples

```
pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)
```

## Evaluation signals

- Precision of the selected pattern equals 1 (no occurrences among any other dataset spectra outside the component)
- Support equals 19 out of 21 component spectra for the best-explaining pattern
- The selected pattern contains an edge with m/z difference 99.0683 annotated with closest formula C5H9ON, consistent with a valine loss
- plotPatterns output confirms the pattern structure is present in 19 of the 21 component spectra
- Summary file records component statistics, pattern edge annotation, support, and precision consistent with the above values

## Limitations

- Exact pattern mining such as mineMS2 extracts a high number of patterns, so candidate filtering is needed before scoring
- A subset of spectrum peaks must be selected due to combinatorial complexity; the current criterion is based solely on peak intensities and may lose chemically relevant information
- No scoring system for patterns yet exists, making best-pattern selection heuristic (coverage/precision over the component)

## Evidence

- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology (Watrous et al., 2012) to focus on patterns that best explain components of the network.: [evidence span withheld — anchor status: not_found]
- [intro] we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra.: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components"
- [supplementary] pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df): "pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)"
- [methods] Closeness checking: The pattern is checked for possible extensions with identical support, to avoid the mining of redundant patterns.: "Closeness checking: The pattern is checked for possible extensions with identical support, to avoid the mining of redundant patterns."
- [discussion] A limitation of exact pattern mining methods such as mineMS2 is the high number of patterns extracted.: "A limitation of exact pattern mining methods such as mineMS2 is the high number of patterns extracted."
