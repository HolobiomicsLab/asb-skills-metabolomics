---
name: spearman-correlation-edge-filtering
description: Use when building metabolite-metabolite network edges from measured concentrations
  or peak intensities (e.g., GC-TOF-MS plasma metabolite data across samples) and
  biochemical pathway or structural annotations are missing, so empirical co-variation
  is the only available association signal.
when_to_use_negative:
- When KEGG RPAIR biochemical transformations, PubChem substructure fingerprints (Tanimoto
  > 0.7), or mass spectral similarities (cosine > 0.7) are available for the analytes
  — those edge types rank above correlation in the unique-edge hierarchy and empirical
  edges may be redundant
- When the input is already a network/edge list rather than a raw metabolite × sample
  intensity matrix — this skill operates on the primary measurement matrix
- When sample size is too small to support a meaningful non-parametric correlation
  test across all metabolite pairs
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Shiny-based R application that calculates the Empirical Dependency Network
    layer and applies the FDR edge-acceptance criterion within the reconstructed metabolic
    network
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Implementation language in which the Spearman correlation matrix, p-value
    computation, and FDR adjustment are carried out
- name: CTSgetR
  role: Optional upstream identifier translation (e.g., chemical names to KEGG or
    PubChem CIDs) needed to place analytes into biochemical/structural network layers
    before correlation edges are merged
  repo: https://github.com/dgrapov/CTSgetR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:62f798087153342ab0fe824ad2815df38dbf4c55128d63dabf27377a66f57dd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - MetaMapR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# spearman-correlation-edge-filtering

## Summary

Computes pairwise metabolite-metabolite dependencies from a metabolite × sample data matrix using non-parametric Spearman correlation (the WGCNA-style empirical dependency network approach of Langfelder and Horvath, 2008), then retains only edges passing FDR-adjusted p-value < 0.05. This supplies empirical association edges for MetaMapR-style networks even when metabolites lack biochemical domain or molecular annotations.

## When to use

Use when building metabolite-metabolite network edges from measured concentrations or peak intensities (e.g., GC-TOF-MS plasma metabolite data across samples) and biochemical pathway or structural annotations are missing, so empirical co-variation is the only available association signal.

## When NOT to use

- When KEGG RPAIR biochemical transformations, PubChem substructure fingerprints (Tanimoto > 0.7), or mass spectral similarities (cosine > 0.7) are available for the analytes — those edge types rank above correlation in the unique-edge hierarchy and empirical edges may be redundant
- When the input is already a network/edge list rather than a raw metabolite × sample intensity matrix — this skill operates on the primary measurement matrix
- When sample size is too small to support a meaningful non-parametric correlation test across all metabolite pairs

## Inputs

- Metabolite data matrix (measured metabolite concentrations or peak intensities across samples) as a delimited/CSV table, one row per analyte
- Optional node attributes table mapping identifiers to analytes

## Outputs

- Filtered edge list CSV (dependency_network_edges.csv) containing only metabolite pairs with FDR-adjusted p-value < 0.05
- Node attributes CSV mapping numeric node IDs (assigned by row order of the uploaded matrix) to uploaded analytes

## How to apply

Load the metabolite data matrix (rows = metabolites, columns = samples) as a delimited table in R. Compute all pairwise metabolite-metabolite Spearman correlations, choosing Spearman over parametric Pearson or biweight because it is non-parametric and robust to outliers in monotonic relationships. Compute p-values for each correlation pair and apply false discovery rate (FDR) adjustment across all pairs. Retain an edge only when the adjusted p-value is < 0.05. Export the accepted edge list together with a node attributes table mapping numeric node IDs to analytes — numeric node IDs are assigned by the row order of the uploaded data (row 1 = node 1) — as comma-separated value files. When combining edge types in MetaMapR, remember correlation edges sit lowest in the unique-edge hierarchy (Biochemical > Structural > Mass Spectral > Correlation) and may be dropped under the unique-edges option.

## Related tools

- **MetaMapR** (Shiny-based R application that calculates the Empirical Dependency Network layer and applies the FDR edge-acceptance criterion within the reconstructed metabolic network) — https://github.com/dgrapov/MetaMapR
- **R** (Implementation language in which the Spearman correlation matrix, p-value computation, and FDR adjustment are carried out)
- **CTSgetR** (Optional upstream identifier translation (e.g., chemical names to KEGG or PubChem CIDs) needed to place analytes into biochemical/structural network layers before correlation edges are merged) — https://github.com/dgrapov/CTSgetR

## Examples

```
m <- as.matrix(read.csv('metabolites.csv', row.names=1)); rho <- cor(t(m), method='spearman'); edges <- rho[p.fdr < 0.05]
```

## Evaluation signals

- Every retained edge in the exported edge list corresponds to a metabolite pair whose correlation passes the edge-acceptance criterion: FDR-adjusted p-value < 0.05
- Numeric node IDs map 1:1 to the row order of the uploaded data matrix (row 1 of the input = node ID 1), verifiable against the node attributes table
- The exported edge list and node attribute mapping are both comma-separated value files importable into network analysis tools such as Cytoscape
- Spearman (non-parametric) correlation was used rather than Pearson/biweight, consistent with the robustness-to-outliers rationale
- Under the unique-edges option, correlation edges are only retained when no higher-priority (Biochemical > Structural > Mass Spectral) edge exists for the same pair

## Limitations

- Correlation edges are the lowest-priority edge type in MetaMapR's unique-edge hierarchy (Biochemical > Structural > Mass Spectral > Correlation), so they can disappear from output when the unique edges checkbox is selected
- The method also supports parametric Pearson and biweight correlations; choosing Pearson may be sensitive to outliers in metabolite intensity distributions
- FDR adjustment is applied across all metabolite pairs simultaneously, so edge retention depends on the total number of tested pairs

## Evidence

- [readme] Unique-edge hierarchy including correlation: [evidence span withheld — anchor status: external_doc]
