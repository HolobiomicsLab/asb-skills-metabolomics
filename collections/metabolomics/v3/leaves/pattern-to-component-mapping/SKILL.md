---
name: pattern-to-component-mapping
description: Use when you have mined mineMS2 patterns, identified which pattern explains
  each GNPS molecular network component, and need to transfer that annotation onto
  the network object and export it for visualization.
when_to_use_negative:
- You only have raw MS/MS spectra and have not yet mined mineMS2 patterns or built
  the GNPS molecular network — run the mining and network-coupling steps first.
- You need patterns attached at the individual-spectrum level rather than aggregated
  per molecular network component.
- You want to propagate existing spectral annotations between adjacent nodes (GNPS's
  built-in behavior) rather than map mined fragmentation patterns onto components.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: mineMS2
  role: Provides the annotateNetwork function that attaches each component's best
    explaining pattern (pattern id, metric values) to the GNPS network object
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Environment in which mineMS2 runs; igraph write.graph interface is used to
    export the annotated network as GraphML
- name: GNPS
  role: Source platform of the molecular network whose components are annotated with
    the best explaining patterns
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:4b7f37fb85d9942cdb216c7531df71bcb49f03b227b27a23f7fef79df19408bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pattern-to-component-mapping

## Summary

Attaches the best-explaining mineMS2 fragmentation pattern to each component of a GNPS molecular network using the mineMS2 annotateNetwork function, then exports the annotated network as GraphML for Cytoscape visualization.

## When to use

Use when you have mined mineMS2 patterns, identified which pattern best explains each GNPS molecular network component, and need to transfer that annotation onto the network object and export it for visualization.

## When NOT to use

- You only have raw MS/MS spectra and have not yet mined mineMS2 patterns or built the GNPS molecular network — run the mining and network-coupling steps first.
- You need patterns attached at the individual-spectrum level rather than aggregated per molecular network component.
- You want to propagate existing spectral annotations between adjacent nodes (GNPS's built-in behavior) rather than map mined fragmentation patterns onto components.

## Inputs

- GNPS molecular network object (loaded into R/mineMS2 from the GNPS molecular networking output)
- best-explaining-pattern annotation table (pattern id, explained component, metric values per component)
- mineMS2 pattern mining results for the spectra collection (e.g. Penicillium nordicum 51-spectrum dataset)

## Outputs

- Annotated GNPS network object with per-component best pattern id and metric values
- GraphML file, e.g. pnordicum_ms2_gnps_mined.graphml, loadable in Cytoscape

## How to apply

Load the GNPS molecular network (from the prior molecular-networking step) and the best-explaining-pattern annotation table (from the prior pattern-mining and pattern-selection steps) into the mineMS2 R package. Apply annotateNetwork to map each component's best explaining pattern onto the network, recording the explained component, the pattern id, and the associated metric values as annotation attributes. Export the annotated network as GraphML via the igraph write.graph interface, e.g. writing pnordicum_ms2_gnps_mined.graphml. This coupling 'focuses on patterns that best explain components' of the GNPS network, aiding interpretation of detected components and annotation of unknown spectra. Validate that the GraphML parses as valid graphml and carries pattern-id and metric attributes on each component, and that it loads in Cytoscape.

## Related tools

- **mineMS2** (Provides the annotateNetwork function that attaches each component's best explaining pattern (pattern id, metric values) to the GNPS network object) — https://github.com/odisce/mineMS2
- **R** (Environment in which mineMS2 runs; igraph write.graph interface is used to export the annotated network as GraphML)
- **GNPS** (Source platform of the molecular network whose components are annotated with the best explaining patterns)

## Examples

```
annotated.net <- annotateNetwork(gnps.net, bestPatterns); igraph::write.graph(annotated.net, "pnordicum_ms2_gnps_mined.graphml", format="graphml")
```

## Evaluation signals

- The output GraphML file exists and parses as a valid graphml document.
- Each network component in the GraphML carries annotation attributes: the best pattern id and the associated metric values.
- The file (e.g. pnordicum_ms2_gnps_mined.graphml) loads successfully in Cytoscape for visualization.
- The number of annotated components matches the number of components in the input GNPS network with a best-explaining pattern.

## Limitations

- mineMS2 exact pattern mining extracts a high number of patterns, so a prior selection of best-explaining patterns per component is required before annotation.
- GNPS molecular network visualization propagates annotations only between pairs and cannot extract common structures shared by more than two spectra; mineMS2 component annotation complements rather than replaces this.
- The annotated output depends on the quality/consistency of the GNPS network, which was built with parameter values selected to match the reference network (Supplementary File 1, Note S2).

## Evidence

- [supplementary] annotateNetwork maps best explaining patterns onto GNPS components: [evidence span withheld — anchor status: not_found]
- [readme] README coupling statement: [evidence span withheld — anchor status: external_doc]
- [intro] Article coupling objective: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
- [supplementary] GraphML export and validation: [evidence span withheld — anchor status: not_found]
