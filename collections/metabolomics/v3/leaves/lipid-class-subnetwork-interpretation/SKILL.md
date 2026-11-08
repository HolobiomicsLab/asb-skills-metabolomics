---
name: lipid-class-subnetwork-interpretation
description: Use when MS/MS networking of differential metabolomics features (e.g.,
  >1000 enriched features) must be resolved into lipid-class subnetworks such as N-acyl
  ethanolamide families for biological interpretation.
when_to_use_negative:
- When the input is raw LC/MS data requiring feature detection first — run the XCMS-based
  Metaboseek feature-detection module before networking
- When metabolites are already fully identified with spectral library matches (Level
  1-2 annotations), making unsupervised clustering of unknowns unnecessary
- When features are not differential between conditions, since subnetwork interpretation
  here is anchored on differential enrichment patterns
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: Metaboseek
  role: Platform for loading feature tables, finding MS2 scans per feature (FindMS2ScansModule),
    viewing/zooming the network graph, and exporting the network as .graphML via SaveNetworkModule
  repo: https://github.com/mjhelf/Metaboseek
- name: MassTools
  role: R package providing makeEdges()/network1() to compute pairwise cosine spectrum
    similarities and build the network edges
  repo: https://github.com/mjhelf/MassTools
- name: igraph
  role: R package underlying network representation/manipulation of the .graphML graph
  repo: https://github.com/igraph/rigraph
- name: xcms
  role: Provides feature detection upstream so that a Feature Table of molecular features
    exists for networking
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:8c1a3f08700aa58c6333739f2fc9072be9bc464260d190b9045a16227f2a195b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MassTools
  - Metaboseek
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lipid-class-subnetwork-interpretation

## Summary

Interprets MS/MS molecular networking results from Metaboseek/MassTools by identifying major subnetworks of related lipid classes and mapping differential enrichment annotations (e.g., hacl-1 exo-metabolome features) onto nodes, yielding class-level chemical hypotheses.

## When to use

Use when MS/MS networking of differential metabolomics features (e.g., >1000 enriched features) must be resolved into lipid-class subnetworks such as N-acyl ethanolamide families for biological interpretation.

## When NOT to use

- When the input is raw LC/MS data requiring feature detection first — run the XCMS-based Metaboseek feature-detection module before networking
- When metabolites are already fully identified with spectral library matches (Level 1-2 annotations), making unsupervised clustering of unknowns unnecessary
- When features are not differential between conditions, since subnetwork interpretation here is anchored on differential enrichment patterns

## Inputs

- Molecular network graph of MS/MS spectra (nodes = molecular features with averaged MS2 scans, edges = cosine similarity scores above the Cosine threshold)
- Differential feature table with enrichment fold-changes (e.g., hacl-1-enriched exo-metabolome features)
- Exported network file molecular_network.graphml (.graphML from SaveNetworkModule)

## Outputs

- Identified subnetworks (e.g., SN1-SN5) assigned to lipid classes: N-acyl glycophosphoethanolamides, N-acyl glycoglycerophosphoethanolamides, N-acyl ethanolamides
- Node annotations mapping lipid class and enrichment magnitude (10-100-fold hacl-1 enrichment) onto graph nodes
- Class-level chemical/biological hypothesis (e.g., alpha-oxidation-dependent N-acyl amide metabolism)

## How to apply

After building the network in Metaboseek with MassTools makeEdges()/network1() — filtering peaks below the 'Noise level in %' intensity threshold, setting a 'min. peaks' match count, optionally enabling 'Use parent masses' so neutral losses contribute, and computing cosine scores between matched-peak intensity vectors — apply a 'Cosine threshold' to retain only significant edges. Inspect the resulting graph in the Metaboseek network viewer (SHIFT+click to zoom into subgraphs, color by default groups) to identify major connected components; in the hacl-1 case the differential features resolved into five subnetworks SN1-SN5 comprising N-acyl glycophosphoethanolamides (SN1), N-acyl glycoglycerophosphoethanolamides (SN2-SN4), and N-acyl ethanolamides (SN5). Annotate nodes with class identities and quantitative enrichment (e.g., the reported 10-100-fold hacl-1 exo-metabolome enrichment for the N-acyl GPE subnetwork) to convert graph topology into lipid-class-level biology.

## Related tools

- **Metaboseek** (Platform for loading feature tables, finding MS2 scans per feature (FindMS2ScansModule), viewing/zooming the network graph, and exporting the network as .graphML via SaveNetworkModule) — https://github.com/mjhelf/Metaboseek
- **MassTools** (R package providing makeEdges()/network1() to compute pairwise cosine spectrum similarities and build the network edges) — https://github.com/mjhelf/MassTools
- **igraph** (R package underlying network representation/manipulation of the .graphML graph) — https://github.com/igraph/rigraph
- **xcms** (Provides feature detection upstream so that a Feature Table of molecular features exists for networking) — https://github.com/sneumann/xcms

## Examples

```
Rscript -e 'library(MassTools); e <- makeEdges(MS2Features, "Noise level in %"=1, "min. peaks"=5, "Cosine threshold"=0.7); g <- network1(e); igraph::write_graph(g, "molecular_network.graphml", format="graphml")'
```

## Evaluation signals

- The exported molecular_network.graphml resolves into five major subnetworks whose composition (N-acyl GPEs, GLEA SN2-SN4, NAEs) matches the reported SN1-SN5 clustering
- N-acyl GPE subnetwork nodes carry the 10-100-fold hacl-1 exo-metabolome enrichment annotation
- All retained edges have cosine scores above the chosen 'Cosine threshold' and connect spectra sharing at least 'min. peaks' matched peaks
- Peaks below the 'Noise level in %' intensity threshold are excluded from similarity computation (no low-intensity peaks contribute to edge scores)
- Majority of the >1000 differential hacl-1-enriched features are assigned to one of the identified lipid-class subnetworks rather than left as singletons

## Limitations

- Subnetwork assignments provide only compound-class-level (confidence Level 3, tentative) annotations, not definitive structures
- Many detected features remain unidentified, so some network nodes cannot be assigned to any lipid class
- The cosine threshold and noise/min-peaks parameters directly shape subnetwork composition; weak edges removed by thresholding can fragment or merge subnetworks

## Evidence

- [results] The majority of the differential hacl-1-enriched exo-metabolome features clustered in five major subnetworks (SN1-SN5), which comprise N-acyl glycophosphoethanolamides (SN1), N-acyl glycoglycerophosphoethanolamides (SN2-SN4), and N-acyl ethanolamides (SN5).: [evidence span withheld — anchor status: not_found]
- [methods] peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded: "peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded"
- [methods] The similarity score will be considered 0 if less than `min. peaks` match between two spectra: "The similarity score
will be considered 0 if less than `min. peaks` match between two spectra"
- [methods] select a threshold for which comparisons to keep (above a given `Cosine threshold`): "select a threshold for which comparisons to keep (above a given 
`Cosine threshold`)"
- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data.: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
