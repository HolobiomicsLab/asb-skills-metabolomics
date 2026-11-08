---
name: marginal-versus-partial-correlation-network-comparison
description: Use when you need to explain or quantify why Pearson-correlation metabolite
  networks are denser than GLASSO-based partial correlation networks built from the
  same peak-intensity table, e.g. reproducing the DNEA paper's MetaBox2 vs DNEA edge-count
  comparison (576 vs 172 edges).
when_to_use_negative:
- When the analysis goal is a biologically interpretable network of direct interactions
  — the article notes marginal networks are dense 'due to the inclusion of edges representing
  both direct and indirect correlations. The latter can confound the results'
- When the dataset has far fewer samples than metabolites and no regularization is
  desired — partial correlation estimation requires regularization (GLASSO) to avoid
  unstable edges in the p >> n regime
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: DNEA R package
  role: Constructs the regularized partial-correlation (GGM/GLASSO) network via joint
    estimation across two experimental groups, with BICtune, stabilitySelection, getNetworks
    and filterNetworks steps
  repo: github.com/Karnovsky-Lab/DNEA
- name: Metabox2
  role: Produces the full pairwise Pearson (marginal) correlation network on the same
    IA metabolomics data for the density comparison
- name: R
  role: Environment for running DNEA, preprocessing (30% missingness filter, median
    imputation), and edge-list export/counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:5b059781d41c697d8a90b08de44399632c6c88b7ac43fc318d543e2035016919
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - Metabox2
  - R
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# marginal-versus-partial-correlation-network-comparison

## Summary

A comparison workflow that builds a marginal (Pearson) correlation network with MetaBox2 and a regularized partial correlation network with DNEA on the same metabolomics data, then quantifies the density difference at a shared |r| >= 0.2 cutoff to demonstrate that marginal networks retain indirect associations.

## When to use

Use when you need to explain or quantify why Pearson-correlation metabolite networks are denser than GLASSO-based partial correlation networks built from the same peak-intensity table, e.g. reproducing the DNEA paper's MetaBox2 vs DNEA edge-count comparison (576 vs 172 edges).

## When NOT to use

- When the analysis goal is a biologically interpretable network of direct interactions — the article notes marginal networks are dense 'due to the inclusion of edges representing both direct and indirect correlations. The latter can confound the results'
- When the dataset has far fewer samples than metabolites and no regularization is desired — partial correlation estimation requires regularization (GLASSO) to avoid unstable edges in the p >> n regime

## Inputs

- Metabolomics peak-intensity table (e.g. TEDDY IA-arm data from Metabolomics Workbench ST001386), samples x metabolites with missing values
- Metabox2 correlation-based network construction settings
- DNEA R package processed dataset object (after createData, BICtune, stabilitySelection, getNetworks)
- Shared edge-strength cutoff (|r| >= 0.2)

## Outputs

- Filtered MetaBox2 Pearson edge list CSV (metabox2_ia_edge_list.csv: Metabolite A, Metabolite B, Pearson r) with 576 edges on TEDDY IA data
- Filtered DNEA partial-correlation edge list via getNetworkFiles()/filterNetworks() with 172 edges at the same cutoff
- Density comparison quantifying the excess of indirect associations in the marginal network

## How to apply

Load the metabolomics peak-intensity data (e.g. TEDDY IA-arm data from Metabolomics Workbench ST001386/PR000950), exclude metabolites missing in more than 30% of samples, and median-impute the remaining missing values. Build the marginal network with MetaBox2's correlation-based construction on the full pairwise Pearson matrix, and the regularized partial correlation network with the DNEA R package, which uses GLASSO with BIC-tuned lambda and stability selection for joint estimation across two conditions. Filter both networks to edges with absolute correlation (or partial correlation) coefficient >= 0.2 — the DNEA filterNetworks(pcor=...) function supports this percentile/threshold filtering. Export each filtered edge list (Metabolite A, Metabolite B, r) and count edges; the marginal network is dense because Pearson correlation links both direct and indirect associations, while partial correlations control for all other metabolites. The expected outcome on the TEDDY IA data is 576 MetaBox2 edges vs 172 DNEA edges at the same cutoff.

## Related tools

- **DNEA R package** (Constructs the regularized partial-correlation (GGM/GLASSO) network via joint estimation across two experimental groups, with BICtune, stabilitySelection, getNetworks and filterNetworks steps) — github.com/Karnovsky-Lab/DNEA
- **Metabox2** (Produces the full pairwise Pearson (marginal) correlation network on the same IA metabolomics data for the density comparison)
- **R** (Environment for running DNEA, preprocessing (30% missingness filter, median imputation), and edge-list export/counting)

## Examples

```
TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.166)
```

## Evaluation signals

- The filtered MetaBox2 IA edge list contains exactly 576 edges at |r| >= 0.2
- The DNEA partial-correlation network contains 172 edges at the same |r| >= 0.2 cutoff
- Preprocessing removes metabolites missing in more than 30% of samples and median-imputes remaining missing values before network construction
- Both networks are filtered with the identical absolute correlation coefficient threshold (0.2), so edge counts are directly comparable
- The marginal network should be markedly denser than the partial-correlation network, since indirect associations are absent from the GLASSO-regularized network

## Limitations

- BIC tuning for DNEA requires a large dataset (~500 or more samples per group); smaller studies must use the lambda approximation (Eq. 3)
- Dense marginal-correlation networks include indirect correlations that can confound identification of biologically relevant interactions
- No software version numbers were provided for DNEA, Filigree, or Metabox2, complicating exact reproduction

## Evidence

- [discussion] marginal networks density: "they tend to be very dense due to the inclusion of edges representing both direct and indirect correlations. The latter can confound the results"
- [discussion] shared 0.2 filtering cutoff: "The DNEA, Filigree and Metabox2 networks were filtered to include edges with an absolute value of correlation coefficient greater than or equal to 0.2."
- [methods] missingness filter: "The data were filtered to exclude metabolites that were missing in more than 30% of the samples."
- [readme] GLASSO partial correlations: "uses the GLASSO algorithm to estimate metabolite-metabolite partial correlations"
