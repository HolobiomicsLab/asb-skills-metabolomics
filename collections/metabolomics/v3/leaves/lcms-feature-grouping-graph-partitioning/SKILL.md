---
name: lcms-feature-grouping-graph-partitioning
description: Use when an xsAnnotate object has already been grouped by retention time
  (groupFWHM) and isotopes (findIsotopes), but compound spectra may still contain
  features of two or more closely co-eluting compounds.
when_to_use_negative:
- The input is not a CAMERA xsAnnotate object pre-processed with groupFWHM and findIsotopes
  — groupCorr expects a retention-time-grouped object with isotope annotation already
  available.
- The goal is initial compound spectrum extraction from raw peaks rather than refinement
  of possibly co-eluting spectra; use groupFWHM-based grouping first.
- No peak-shape or cross-sample intensity-correlation information is available (e.g.
  single-sample data without replicates), making calcCiS/calcCaS edge scores uninformative.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Provides the groupCorr function that builds the weighted relationship graph
    and invokes calcPC.hcs or calcPC.lpc to split compound spectra
  repo: https://github.com/sneumann/CAMERA
- name: RBGL
  role: Supplies the highly connected subgraphs (calcPC.hcs) partitioning algorithm
- name: igraph
  role: Supplies the label propagation (calcPC.lpc) partitioning algorithm
- name: graph
  role: Graph data structure used to hold the weighted relationship edges
- name: xcms
  role: Upstream peak detection that produces the peak features grouped by CAMERA
- name: R
  role: Implementation language for CAMERA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:b5483a0e9fb8242d63cb28c33cdf13ab7c4bc315cc95aafbd6387b01afa11a96
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - Biobase
  - CAMERA
  - R
  - RBGL
  - graph
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lcms-feature-grouping-graph-partitioning

## Summary

Refines LC/MS compound spectra that may still contain features of two or more closely co-eluting compounds by building a weighted relationship graph of cross-sample and peak-shape correlation edges and splitting it with highly connected subgraph (HCS) or label propagation (LPC) partitioning, as implemented in CAMERA's groupCorr.

## When to use

Use when an xsAnnotate object has already been grouped by retention time (groupFWHM) and isotopes (findIsotopes), but compound spectra may still contain features of two or more closely co-eluting compounds.

## When NOT to use

- The input is not a CAMERA xsAnnotate object pre-processed with groupFWHM and findIsotopes — groupCorr expects a retention-time-grouped object with isotope annotation already available.
- The goal is initial compound spectrum extraction from raw peaks rather than refinement of possibly co-eluting spectra; use groupFWHM-based grouping first.
- No peak-shape or cross-sample intensity-correlation information is available (e.g. single-sample data without replicates), making calcCiS/calcCaS edge scores uninformative.

## Inputs

- CAMERA xsAnnotate object already processed with groupFWHM and findIsotopes
- Raw LC/MS peak intensity data from which extracted ion chromatograms (EICs) are computed
- Parameters: cor_eic_th, pval, graphMethod ('hcs' or 'lpc'), calcIso, calcCaS

## Outputs

- Refined pseudospectra assignment splitting co-eluting compound spectra
- Annotated peak list exported via getPeaklist (e.g. groupcorr_peaklist.tsv, tab-separated)

## How to apply

Load the CAMERA xsAnnotate object (already processed with groupFWHM and findIsotopes) and call groupCorr with cor_eic_th for the cross-sample intensity-correlation threshold, pval for the correlation significance cutoff, and graphMethod set to 'hcs' or 'lpc'. Pass calcIso=TRUE to include the isotope relation in the graph and calcCaS=TRUE to include peak-shape (calcCaS) correlation in the edge scores. Internally, groupCorr computes EICs via getAllPeakEICs, scores edges with calcCiS (cross-sample correlation, parameters corval and pval) and calcCaS (peak-shape correlation), merges these with isotope-relation edges into a combined weighted graph, and splits each compound spectrum via calcPC.hcs (RBGL highly connected subgraphs) or calcPC.lpc (igraph label propagation). Export the refined pseudospectra with getPeaklist as a tab-separated table (groupcorr_peaklist.tsv).

## Related tools

- **CAMERA** (Provides the groupCorr function that builds the weighted relationship graph and invokes calcPC.hcs or calcPC.lpc to split compound spectra) — https://github.com/sneumann/CAMERA
- **RBGL** (Supplies the highly connected subgraphs (calcPC.hcs) partitioning algorithm)
- **igraph** (Supplies the label propagation (calcPC.lpc) partitioning algorithm)
- **graph** (Graph data structure used to hold the weighted relationship edges)
- **xcms** (Upstream peak detection that produces the peak features grouped by CAMERA)
- **R** (Implementation language for CAMERA)

## Examples

```
R: groupCorr(xsAnnotate, cor_eic_th=0.75, pval=0.05, graphMethod='hcs', calcIso=TRUE, calcCaS=TRUE); getPeaklist(xsAnnotate, file='groupcorr_peaklist.tsv')
```

## Evaluation signals

- On a synthetic example where two co-eluting compounds are simulated, the two compounds must fall into different pseudospectra (the graph is correctly split at the weak edge between them).
- Setting calcIso=TRUE must result in isotope-relation edges being merged into the combined weighted graph before partitioning.
- Choosing graphMethod='lpc' vs 'hcs' must route the split through calcPC.lpc (igraph) vs calcPC.hcs (RBGL), respectively.
- getPeaklist on the refined object must produce a tab-separated table whose pseudospectra assignment reflects the refined grouping (groupcorr_peaklist.tsv).
- cor_eic_th and pval must be applied as the cross-sample intensity-correlation threshold and correlation significance cutoff in the calcCiS edge scores.

## Limitations

- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- CAMERA does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion; annotation relies on the dynamic rule table.
- Highly complex data may still leave some compound spectra under-explained, since annotations occurred either because the compound spectrum did not contain enough explained features, or in other cases several hypotheses had the same precedence scores.

## Evidence

- [methods] CAMERA builds a relationship graph in which the features of a possibly co-eluting compound spectrum are nodes connected by edges weighted with a combined score, and separates the spectrum using the Highly-connected-subgraphs (HCS) algorithm.: [evidence span withheld — anchor status: not_found]
- [methods] groupCorr internally computes EICs, scores cross-sample correlation edges with calcCiS and peak-shape correlation edges with calcCaS, and splits each compound spectrum by calling calcPC with either calcPC.hcs or calcPC.lpc as chosen by graphMethod.: [evidence span withheld — anchor status: not_found]
