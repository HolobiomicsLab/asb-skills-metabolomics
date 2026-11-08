---
name: biweight-midcorrelation-computation
description: Use when a metabolite-by-sample data matrix (concentrations or peak intensities,
  e.g. from GC-TOF-MS) must yield empirical correlation edges for network reconstruction,
  especially when biochemical/domain annotations are unknown.
when_to_use_negative:
- The metabolites already have complete KEGG RPAIR annotations and the goal is purely
  biochemical reaction edges — use Biochemical Reaction Networks instead of empirical
  correlation.
- The data matrix has too few samples to support stable pairwise correlation estimates,
  or the input is already an edge list rather than a sample-wise measurement matrix.
- The target association is structural (PubChem substructure fingerprint Tanimoto
  similarity >0.7) or spectral (cosine correlation >0.7) rather than sample-co-variation.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaMapR
  role: Implements the Empirical Dependency Networks calculation (Langfelder and Horvath,
    2008) with Pearson, biweight, and Spearman correlation options as one of four
    edge types
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which the empirical dependency (biweight/Spearman)
    correlation computation is implemented; the data matrix is loaded as a delimited
    table in R
  repo: http://cran.us.r-project.org/
- name: Shiny
  role: R package powering the MetaMapR user interface through which correlation method,
    edge thresholds, and unique-edge options are selected
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:1c0fb8dcc1e58f9a3e3d678a65a9787a8924170268c84eeb0db5b4d80af93db0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# biweight-midcorrelation-computation

## Summary

Computes pairwise metabolite-metabolite empirical dependency edges from a metabolite data matrix using the WGCNA-style correlation approach of Langfelder and Horvath (2008), with biweight midcorrelation (robust M-estimator) as the parametric option alongside Pearson and non-parametric Spearman correlations, as implemented in MetaMapR's Empirical Dependency Networks module.

## When to use

Use when a metabolite-by-sample data matrix (concentrations or peak intensities, e.g. from GC-TOF-MS) must yield empirical correlation edges for network reconstruction, especially when biochemical/domain annotations are unknown.

## When NOT to use

- The metabolites already have complete KEGG RPAIR annotations and the goal is purely biochemical reaction edges — use Biochemical Reaction Networks instead of empirical correlation.
- The data matrix has too few samples to support stable pairwise correlation estimates, or the input is already an edge list rather than a sample-wise measurement matrix.
- The target association is structural (PubChem substructure fingerprint Tanimoto similarity >0.7) or spectral (cosine correlation >0.7) rather than sample-co-variation.

## Inputs

- Metabolite data matrix: measured metabolite concentrations or peak intensities across samples, as a delimited table loadable in R
- Choice of correlation method (parametric Pearson, biweight, or non-parametric Spearman)
- Optional significance threshold for edge acceptance (e.g. FDR-adjusted p-value <0.05)

## Outputs

- Dependency network edge list of metabolite pairs meeting the correlation significance criterion (e.g. comma-separated dependency_network_edges.csv)
- Node attributes file mapping numeric node IDs (row order of the uploaded matrix) to uploaded analytes
- Edges incorporable into the combined MetaMapR network and exportable to Cytoscape

## How to apply

Load the metabolite data matrix (analytes as rows, samples as columns) as a delimited table in R and compute all pairwise metabolite-metabolite dependencies using one of the Langfelder and Horvath (2008) correlation options: parametric Pearson, robust biweight midcorrelation, or non-parametric Spearman — biweight being preferred when outliers could distort Pearson coefficients. In MetaMapR these empirical edges are one of four orthogonal association measures combined with biochemical (KEGG RPAIR), structural (PubChem substructure fingerprint Tanimoto), and mass spectral (cosine) edges. Apply a significance filter to the correlation pairs (e.g. FDR-adjusted p-value <0.05) before accepting edges, then export the edge list with node attributes mapping numeric node IDs (assigned by row order of the uploaded matrix) back to analyte names. Note that MetaMapR's unique-edges option applies the hierarchy Biochemical > Structural > Mass Spectral > Correlation, so correlation (including biweight-derived) edges are retained only when no higher-priority edge exists between the pair.

## Related tools

- **MetaMapR** (Implements the Empirical Dependency Networks calculation (Langfelder and Horvath, 2008) with Pearson, biweight, and Spearman correlation options as one of four edge types) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which the empirical dependency (biweight/Spearman) correlation computation is implemented; the data matrix is loaded as a delimited table in R) — http://cran.us.r-project.org/
- **Shiny** (R package powering the MetaMapR user interface through which correlation method, edge thresholds, and unique-edge options are selected)

## Evaluation signals

- Every retained edge in dependency_network_edges.csv corresponds to a metabolite pair whose correlation meets the edge-acceptance criterion (FDR-adjusted p-value <0.05).
- Numeric node IDs in the edge list map 1:1 to the row order of the uploaded data matrix (node attributes tab confirms the identifier-to-numeric-ID mapping).
- If the unique-edges option was selected, no pair carries both a higher-priority edge (Biochemical > Structural > Mass Spectral) and a correlation edge — the hierarchy was respected.
- Correlations were computed across the same sample set for all pairs, and the correlation method (biweight vs Pearson vs Spearman) matches what was selected in the network calculation options.

## Limitations

- Biweight is one of several correlation options; the non-parametric Spearman alternative is more robust to outliers and monotonic (non-linear) relationships, so results differ by choice.
- In MetaMapR's unique-edges mode, correlation edges are deprioritized below biochemical, structural, and mass spectral edges, so many biweight-derived edges may be dropped from the final network.
- Empirical correlations alone can connect metabolites lacking biochemical intermediates or domain knowledge, but the article notes such correlations are one orthogonal measure and do not confer biochemical directionality (a limitation the paper attributes to KEGG for biochemical edges).
- Edge acceptance depends on the FDR significance threshold; varying the threshold changes network connectivity.

## Evidence

- [methods] Empirical Dependency Networks are calculated (Langfelder and Horvath, 2008) based on the parametric Pearson and biweight correlations or non-parametric Spearman correlations: "Empirical Dependency Networks are calculated (Langfelder and Horvath, 2008) based on the parametric Pearson and biweight correlations or non-parametric Spearman correlations"
- [abstract] MetaMapR integrates enzymatic transformations with metabolite structural similarity, mass spectral similarity and empirical associations to generate richly connected metabolic networks.: "MetaMapR integrates enzymatic transformations with metabolite structural similarity, mass spectral similarity and empirical associations to generate richly connected metabolic networks."
- [readme] By default the numeric node IDs are assigned based on the order (row) of the uploaded data. For example 1 corresponds to the analyte in the first row.: "By default the numeric node IDs are assigned based on the order (row) of the uploaded data. For example 1 corresponds to the analyte in the first row."
- [readme] The default setting for returning all edge types or only unique edges can be found under the network tab using the unique edges checkbox. If the option is selected then edges are returned based on a hierarchy Biochemical > Structural > Mass Spectral > Correlation: [evidence span withheld — anchor status: external_doc]
