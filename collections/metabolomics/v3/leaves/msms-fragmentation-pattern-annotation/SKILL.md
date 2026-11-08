---
name: msms-fragmentation-pattern-annotation
description: Use when a GNPS molecular network (MS/MS spectra as vertices, modified-cosine
  edges) must be interpreted by mapping mineMS2-mined fragmentation patterns onto
  its components and exporting for Cytoscape.
when_to_use_negative:
- You have only a spectral library or single MS/MS spectra to mine — use the mineMS2
  pattern-mining pipeline (discretizeMzDifferences, fragmentation-graph building)
  without GNPS coupling
- Your network already carries substructure annotations from an alternative approach
  such as MS2LDA/MolNetEnhancer and you do not want mineMS2-exact-pattern annotations
- You need propagation of annotations between neighbouring spectra rather than per-component
  best-explaining patterns (GNPS visualization already handles pairwise annotation
  propagation)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mineMS2
  role: R package whose annotateNetwork function maps each component's best explaining
    pattern (explained component, pattern id, metric values) onto the GNPS network
    object
  repo: https://github.com/odisce/mineMS2
- name: igraph
  role: Provides the write.graph interface used to export the annotated network as
    a GraphML file
- name: GNPS
  role: Source platform of the MS/MS molecular network (vertices = spectra, edges
    = modified cosine score) that is being annotated
- name: Cytoscape
  role: Target visualization tool for the exported GraphML annotated network
- name: R
  role: Runtime environment in which mineMS2 (Rcpp/C++ wrapped) and the GraphML export
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:b5605af6c5b443aa94f8f970d350cb27b42ebefb12da44af38c60852d51b3b06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - Cytoscape
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msms-fragmentation-pattern-annotation

## Summary

Attach the best-explaining mineMS2 fragmentation pattern to each component of a GNPS molecular network and export the annotated network as GraphML for Cytoscape visualization, coupling exact graph-structured patterns to molecular networking.

## When to use

Use when a GNPS molecular network (MS/MS spectra as vertices, modified-cosine edges) must be interpreted by mapping mineMS2-mined fragmentation patterns onto its components and exporting for Cytoscape.

## When NOT to use

- You have only a spectral library or single MS/MS spectra to mine — use the mineMS2 pattern-mining pipeline (discretizeMzDifferences, fragmentation-graph building) without GNPS coupling
- Your network already carries substructure annotations from an alternative approach such as MS2LDA/MolNetEnhancer and you do not want mineMS2-exact-pattern annotations
- You need propagation of annotations between neighbouring spectra rather than per-component best-explaining patterns (GNPS visualization already handles pairwise annotation propagation)

## Inputs

- GNPS molecular network object for the MS/MS collection (vertices = spectra, edges = modified cosine similarity), e.g. the *Penicillium nordicum* network from the 51-spectrum MGF
- Best-explaining-pattern annotation table from mineMS2 mining, mapping molecular network components to pattern ids and metric values
- Spectral metadata TSV (e.g. dataset/pnordicum_ms2_info.tsv supplied with the mineMS2 package)

## Outputs

- Annotated molecular network as a GraphML file (e.g. pnordicum_ms2_gnps_mined.graphml) with pattern id and metric-value attributes attached to each network component
- Component-to-pattern mapping (explained component, pattern id, metric values) held on the igraph network object

## How to apply

Load two artefacts into the mineMS2 R package: the GNPS molecular network object (e.g. from the *Penicillium nordicum* 51-spectrum MGF plus metadata such as dataset/pnordicum_ms2_info.tsv) and the best-explaining-pattern table produced by the earlier pattern-to-component mining step. Apply mineMS2::annotateNetwork to map each component to its best explaining pattern, recording the explained component, the pattern id, and the associated metric values (e.g. F1-score-style support metrics used to rank patterns). Then export the annotated igraph network with igraph::write.graph in GraphML format, e.g. as pnordicum_ms2_gnps_mined.graphml. Rationale: mineMS2 patterns are exact fragmentation graphs of m/z differences (including neutral losses and cross-path m/z differences), so annotating components with pattern ids propagates structural interpretation beyond pairwise GNPS cosine-similarity edges.

## Related tools

- **mineMS2** (R package whose annotateNetwork function maps each component's best explaining pattern (explained component, pattern id, metric values) onto the GNPS network object) — https://github.com/odisce/mineMS2
- **igraph** (Provides the write.graph interface used to export the annotated network as a GraphML file)
- **GNPS** (Source platform of the MS/MS molecular network (vertices = spectra, edges = modified cosine score) that is being annotated)
- **Cytoscape** (Target visualization tool for the exported GraphML annotated network)
- **R** (Runtime environment in which mineMS2 (Rcpp/C++ wrapped) and the GraphML export are executed)

## Examples

```
annotated_g <- mineMS2::annotateNetwork(gnps_network, best_patterns); igraph::write.graph(annotated_g, "pnordicum_ms2_gnps_mined.graphml", format = "graphml")
```

## Evaluation signals

- The output GraphML file exists and parses as a valid GraphML document
- Each network component carries annotation attributes including the pattern id and the associated metric values
- The annotated file loads correctly in Cytoscape and displays components with pattern annotations
- The component-to-pattern mapping matches the best-explaining-pattern table computed in the preceding mining step (same pattern ids per component)
- Number of annotated components is consistent with the number of components in the loaded GNPS network object

## Limitations

- mineMS2 exact pattern mining extracts a high number of patterns, so the best-explaining-pattern selection step feeds the annotation and can dominate the result
- Patterns capture similarities only among spectra sharing the exact m/z-difference graph; GNPS visualization itself propagates annotations pairwise and cannot extract structures shared by more than two spectra, which is the gap this annotation step addresses
- Interpretation of annotated patterns is manual: no scoring system or database of patterns of interest with a dedicated query metric exists yet (proposed as future work)

## Evidence

- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking: [evidence span withheld — anchor status: external_doc]
- [abstract] a software library enabling efficient mining of MS/MS data and coupling to molecular networks: "a software library enabling efficient mining of MS/MS data and coupling to molecular networks"
- [intro] we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra.: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
- [intro] In molecular networks, as generated by the GNPS platform, each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score: "In molecular networks, as generated by the GNPS platform [27], each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score"
- [readme] vignette detailing how to focus on the patterns that best explain components of the molecular network: [evidence span withheld — anchor status: external_doc]
