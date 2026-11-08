---
name: msms-pattern-interpretation
description: Use when a GNPS molecular network component or spectral cluster needs
  a chemical interpretation, and MS/MS spectra (e.g., LC-HRMS .mgf with metadata)
  can be mined for shared, graph-structured m/z differences.
when_to_use_negative:
- The collection contains fewer than two spectra per pattern's support threshold —
  mineMS2 requires m/z differences frequent in at least two spectra, so single-spectrum-per-compound
  libraries cannot yield patterns
- Only low-resolution MS2 or fragment/loss differences outside the 14–200 Da formula-generation
  window are available — candidate molecular formulas cannot be computed for interpretation
- The goal is a probabilistic distribution of fragments/losses (MS2LDA-style motifs)
  rather than exact graph-structured patterns whose every m/z difference is present
  in all containing spectra
edam_operation: http://edamontology.org/operation_3357
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: mineMS2
  role: R package providing ms2Lib (spectrum loading), infoPatterns (pattern support/precision
    over components), and plotPatterns (pattern visualization) to mine and interpret
    fragmentation patterns
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components are explained by mined patterns;
    edges indicate high pairwise modified cosine similarity
- name: igraph
  role: R graph library used to identify the largest connected component (e.g., 21
    spectra) of the GNPS network
- name: R
  role: Host environment for the mineMS2 Rcpp/C++ package and the interpretation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:eff3d1132c45e1737bc2ef60a33b9473b8ea35cbc158ee1f25cbf0650a204e9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: http://edamontology.org/operation_3357
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# msms-pattern-interpretation

## Summary

Interprets exact fragmentation patterns mined by mineMS2 from MS/MS spectra collections by annotating pattern edges with candidate neutral formulas and coupling the patterns to GNPS molecular network components. This turns graph-structured m/z differences (e.g., 99.0683 Da, closest formula C5H9ON, a valine loss) into chemical hypotheses explaining shared fragmentation within groups of spectra.

## When to use

Use when a GNPS molecular network component or spectral cluster needs a chemical interpretation, and MS/MS spectra (e.g., LC-HRMS .mgf with metadata) can be mined for shared, graph-structured m/z differences.

## When NOT to use

- The collection contains fewer than two spectra per pattern's support threshold — mineMS2 requires m/z differences frequent in at least two spectra, so single-spectrum-per-compound libraries cannot yield patterns
- Only low-resolution MS2 or fragment/loss differences outside the 14–200 Da formula-generation window are available — candidate molecular formulas cannot be computed for interpretation
- The goal is a probabilistic distribution of fragments/losses (MS2LDA-style motifs) rather than exact graph-structured patterns whose every m/z difference is present in all containing spectra

## Inputs

- MS/MS spectrum library in .mgf format (e.g., pnordicum.mgf from P. nordicum secondary metabolites, 51 spectra)
- Spectrum metadata TSV (e.g., pnordicum_ms2_info.tsv) passed as suppInfos to ms2Lib
- GNPS molecular network component (vertex = spectrum, edge = high modified cosine pairwise similarity) loaded into R/igraph
- Mined mineMS2 pattern set from a prior frequent-subgraph mining run

## Outputs

- Best-explaining pattern per network component with support (e.g., 19/21 spectra) and precision (e.g., 1)
- Pattern edge annotation: m/z difference (e.g., 99.0683) with closest candidate molecular formula (e.g., C5H9ON) and chemical interpretation (valine loss)
- Pattern structure plots mapped onto component spectra (plotPatterns)
- Summary file of component statistics, pattern edge annotations, support, and precision

## How to apply

Represent each MS/MS spectrum as a fragmentation graph whose vertices are ion peaks and whose edges are binned m/z differences (disjoint bins derived from differences between the 15 most intense fragment ions, with a 15 ppm tolerance selected because lower values were too sensitive to small m/z variations). Run the frequent-subgraph mining pipeline — candidate graph generation, frequency testing, and closeness checking to avoid redundant patterns — and generate possible neutral formulas for m/z differences between 14 and 200 Da, discarding differences with no possible formula. Load the GNPS molecular network (igraph in R), identify the target component, and use the mineMS2 infoPatterns function to compute support and precision of each pattern over the component's spectra, selecting the pattern with the highest coverage and precision. Verify the winning pattern's key edge (e.g., m/z difference 99.0683, closest formula C5H9ON, consistent with a valine loss) and visualize with plotPatterns mapped onto the component spectra. Confirm the pattern occurs in the expected number of component spectra (e.g., 19 of 21) and check occurrences across the full dataset to ensure precision of 1 (no occurrences elsewhere), then save component statistics, edge annotations, support, and precision in a summary file.

## Related tools

- **mineMS2** (R package providing ms2Lib (spectrum loading), infoPatterns (pattern support/precision over components), and plotPatterns (pattern visualization) to mine and interpret fragmentation patterns) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components are explained by mined patterns; edges indicate high pairwise modified cosine similarity)
- **igraph** (R graph library used to identify the largest connected component (e.g., 21 spectra) of the GNPS network)
- **R** (Host environment for the mineMS2 Rcpp/C++ package and the interpretation workflow)

## Examples

```
pnordicum_metadata.df <- read.table(system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")); pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)
```

## Evaluation signals

- The pattern's key edge is annotated with the expected m/z difference and closest formula (e.g., 99.0683 ↔ C5H9ON, valine loss)
- Precision equals 1: the pattern has no occurrences in dataset spectra outside the target component
- Support matches the expected fraction of the component (e.g., 19 of 21 spectra in the fungisporin cluster)
- All annotated m/z differences fall in the 14–200 Da formula-generation window and were binned with the 15 ppm tolerance
- Closeness checking was applied so that the reported pattern has no extension with identical support (no redundant pattern)

## Limitations

- Exact pattern mining such as mineMS2 extracts a high number of patterns, and no scoring system for patterns yet exists (proposed as future work)
- A subset of spectrum peaks must be selected due to combinatorial complexity; the current criterion is based solely on peak intensities and may lose chemically relevant information
- The DAG representation does not use knowledge of the precursor molecular formula (this option could be added in future versions of the package)
- No database of patterns of interest with a dedicated query metric yet exists, so structural interpretation of patterns is manual

## Evidence

- [abstract] these new fragmentation patterns capture similarities that are not extracted by existing methods, and facilitate the structural interpretation of molecular network components: "these new fragmentation patterns capture similarities that are not extracted by existing methods, and facilitate the structural interpretation of molecular network components"
- [methods] Possible neutral formulas are generated for all the m/z differences between 14 and 200.: "Formula generation:  Possible neutral formulas are generated for all the m/z differences between 14 and 200."
- [results] We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [readme] to focus on patterns that best explain components of the network.: [evidence span withheld — anchor status: external_doc]
- [intro] the detected patterns, which are distributions of fragment and losses from the precursor, cannot be easily related to the underlying physical fragmentation process: "the detected patterns, which are distributions of fragment and losses from the precursor, cannot be easily related to the underlying physical fragmentation process"
