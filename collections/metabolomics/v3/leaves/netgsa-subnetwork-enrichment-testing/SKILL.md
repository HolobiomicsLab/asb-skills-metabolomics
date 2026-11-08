---
name: netgsa-subnetwork-enrichment-testing
description: Use when two-condition metabolomics networks have been estimated with
  DNEA (glasso partial correlations + clusterNet consensus clustering) and you need
  to determine which metabolic subnetworks are differentially enriched between groups.
when_to_use_negative:
- When all metabolites can be confidently mapped to well-annotated KEGG pathways and
  standard reference-based pathway enrichment is sufficient — data-driven module testing
  is designed for datasets containing exogenous, secondary, and unknown metabolites.
- When the network has not yet been clustered into subnetworks (no clusterNet output)
  — NetGSA enrichment requires defined metabolic modules with >= 5 metabolites.
- When analyzing a single experimental group with no case/control contrast — NetGSA
  here tests differential enrichment between two conditions.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: DNEA R package
  role: Provides runNetGSA() to test clustered subnetworks (min_size = 5) for differential
    enrichment and getNetworkFiles() to export node/edge lists
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: netgsa
  role: Underlying algorithm used by runNetGSA to test data-driven metabolic modules
    for enrichment across experimental conditions
- name: Cytoscape v3.10.0
  role: Visualization of the enriched subnetworks by importing the edge list generated
    with getNetworkFiles()
- name: igraph
  role: Clustering algorithms (7 of them) used in clusterNet to define the subnetworks
    that are subsequently enriched-tested
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:9460357c5e6ccf4debf41d08dcead13e6c4e418712e7dc2fc6045be0a028ea8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - Cytoscape v3.10.0
  - DNEA R package
  - R
  - glasso
  - igraph
  - netgsa
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# netgsa-subnetwork-enrichment-testing

## Summary

Tests data-driven metabolic subnetworks (modules) identified by DNEA consensus clustering for differential enrichment between two experimental groups using the NetGSA algorithm, as implemented in the DNEA R package's runNetGSA() function. This replaces reference-based pathway analysis when metabolites (e.g., exogenous, lipid, or unknown compounds) cannot be mapped to pathway databases like KEGG.

## When to use

Use when two-condition metabolomics networks have been estimated with DNEA (glasso partial correlations + clusterNet consensus clustering) and you need to determine which metabolic subnetworks are differentially enriched between groups.

## When NOT to use

- When all metabolites can be confidently mapped to well-annotated KEGG pathways and standard reference-based pathway enrichment is sufficient — data-driven module testing is designed for datasets containing exogenous, secondary, and unknown metabolites.
- When the network has not yet been clustered into subnetworks (no clusterNet output) — NetGSA enrichment requires defined metabolic modules with >= 5 metabolites.
- When analyzing a single experimental group with no case/control contrast — NetGSA here tests differential enrichment between two conditions.

## Inputs

- DNEA object (e.g., TEDDYdat) after getNetworks() and clusterNet(), containing condition-specific glasso networks and subnetwork membership
- TEDDY T1D metabolomics case-vs-control data (134 metabolites, 322 samples; Metabolomics Workbench ST001386/ST001636) as a concrete example input
- Minimum subnetwork size threshold (min_size = 5 metabolites)

## Outputs

- Per-subnetwork NetGSA differential enrichment statistics with FDR-adjusted p-values (exported as teddy_netgsa_results.csv in the reference workflow)
- Ranked list of significant metabolic modules (e.g., S1–S8 significant at FDR < 0.05 out of 11 testable subnetworks)
- Cytoscape-ready node and edge lists from getNetworkFiles() for visualizing enriched modules

## How to apply

After getNetworks() estimates condition-specific glasso networks and clusterNet(tau = 0.5, max_iterations = 5) assigns metabolites to subnetworks via 7 igraph clustering algorithms, run runNetGSA(min_size = 5) in the DNEA R package to test every subnetwork containing at least 5 metabolites for differential enrichment with the NetGSA algorithm. NetGSA uses the data-driven networks themselves as the pathway definition, so no KEGG or other pathway ID mapping is needed — a key advantage since, for the TEDDY data, FELLA could annotate only 19 of 174 metabolites. Rank resulting subnetworks by FDR-adjusted significance (e.g., in the TEDDY case-vs-control analysis, 8 of 11 testable subnetworks, S1–S8, were significant at FDR < 0.05, with S1 top-ranked). Optionally filter weak edges first (e.g., filterNetworks(pcor = 0.166)) before clustering to control which modules are tested.

## Related tools

- **DNEA R package** (Provides runNetGSA() to test clustered subnetworks (min_size = 5) for differential enrichment and getNetworkFiles() to export node/edge lists) — http://www.github.com/Karnovsky-Lab/DNEA/
- **netgsa** (Underlying algorithm used by runNetGSA to test data-driven metabolic modules for enrichment across experimental conditions)
- **Cytoscape v3.10.0** (Visualization of the enriched subnetworks by importing the edge list generated with getNetworkFiles())
- **igraph** (Clustering algorithms (7 of them) used in clusterNet to define the subnetworks that are subsequently enriched-tested)

## Examples

```
TEDDYdat <- runNetGSA(object = TEDDYdat, min_size = 5); getNetworkFiles(TEDDYdat)
```

## Evaluation signals

- The number of testable subnetworks equals those with >= 5 metabolites of the clustered features (TEDDY reference: 11 testable of 13 clustered)
- Expected enrichment outcome for TEDDY case-vs-control: 8 of 11 subnetworks (S1–S8) significant at FDR < 0.05, with subnetwork 1 top-ranked by FDR
- teddy_netgsa_results.csv is produced and parses correctly with per-subnetwork FDR values
- Exported node and edge list files from getNetworkFiles() exist and import successfully into Cytoscape

## Limitations

- Only subnetworks with at least 5 metabolites can be tested; smaller modules are excluded from enrichment analysis.
- Downstream interpretation inherits upstream network-quality issues: network sparsity increases as sample size decreases (e.g., T1D: 26,888 total edges at 100% of samples vs 1354 at 5%), which affects module composition.
- Enrichment results depend on the consensus-clustering result (tau = 0.5, max_iterations = 5); unstable clustering propagates to unstable module significance.

## Evidence

- [intro] Subnetwork enrichment testing using the NetGSA algorithm: "The final step of the DNEA algorithm tests the resulting subnetworks, or metabolic modules, for enrichment using the NetGSA algorithm"
- [results] Eleven testable subnetworks, eight significant: [evidence span withheld — anchor status: not_found]
- [methods] runNetGSA invocation: [evidence span withheld — anchor status: not_found]
- [discussion] KEGG annotation limits motivation: "FELLA requires KEGG IDs, limiting the analysis to the metabolites that are annotated in KEGG. As a result, only 19 out of 174 metabolites present in the TEDDY data were included"
- [readme] netgsa role in workflow: "Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test them for enrichment across experimental conditions using the"
