---
name: iokr-kernel-regression-link-scoring
description: Use when you have paired microbial genome-mining (AntiSMASH BGC/GCF)
  and metabolomics (GNPS spectral) data loaded in NPLinker and need a kernel-regression
  score to rank candidate BGC–spectrum links, e.g. against a validated link subset.
when_to_use_negative:
- The goal is not link scoring but data preparation, e.g. GNPS download/extraction,
  AntiSMASH BGC preparation, or BigScape GCF clustering, which NPLinker provides as
  separate upstream steps
- No paired genomics and metabolomics data are available for the strains, so candidate
  BGC–spectrum links cannot be constructed
- You need a validated standardised hypergeometric-style score rather than the novel
  kernel-regression score; combine IOKR with complementary scoring functions instead
  of using it alone as ground truth
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NPLinker
  role: Framework in which the IOKR scoring method is invoked via npl.get_links()
    to compute and score candidate genomic–metabolomic links
  repo: https://github.com/NPLinker/nplinker
- name: nplinker
  role: Python package providing load_data(), get_links(), LinkGraph.links and LinkGraph.get_link_data()
    used to extract per-link IOKR scores
  repo: https://github.com/NPLinker/nplinker
- name: AntiSMASH
  role: Supplies the BGC data in the antismash/ directory that constitute one side
    of the scored links
- name: BigScape
  role: Clusters AntiSMASH BGCs into GCFs (run automatically by NPLinker if bigscape/
    output is absent) used as link endpoints
- name: GNPS
  role: Source of the molecular networking data whose MS2 spectra form the metabolomic
    side of the scored links
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:f1b38b769d7a00f5e6974db780469f92fab77fa40c95e2aef5136273d87b9253
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - AntiSMASH
  - BigScape
  - GNPS
  - NPLinker
  - Python
  - nplinker
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# iokr-kernel-regression-link-scoring

## Summary

Applies NPLinker's Input-Output Kernel Regression (IOKR) scoring method to prioritise links between BGC/GCF objects and MS2 spectra from integrated genomics–metabolomics data, producing per-link scores that can be compared against literature-validated links.

## When to use

Use when you have paired microbial genome-mining (AntiSMASH BGC/GCF) and metabolomics (GNPS spectral) data loaded in NPLinker and need a kernel-regression score to rank candidate BGC–spectrum links, e.g. against a validated link subset.

## When NOT to use

- The goal is not link scoring but data preparation, e.g. GNPS download/extraction, AntiSMASH BGC preparation, or BigScape GCF clustering, which NPLinker provides as separate upstream steps
- No paired genomics and metabolomics data are available for the strains, so candidate BGC–spectrum links cannot be constructed
- You need a validated standardised hypergeometric-style score rather than the novel kernel-regression score; combine IOKR with complementary scoring functions instead of using it alone as ground truth

## Inputs

- NPLinker working directory with nplinker.toml (root_dir; gnps/, antismash/, bigscape/ subdirectories and strain_mappings.json)
- GNPS molecular networking data (spectra and molecular families)
- AntiSMASH BGC data clustered into GCFs (directly or via automatic BigScape run)
- strain_mappings.json mapping strains to genomics and metabolomics data
- MIBiG metadata (automatically downloaded by NPLinker) for structure assignment

## Outputs

- LinkGraph of scored BGC/GCF–spectrum links with per-link IOKR scores
- Per-link score table (e.g. iokr_scores.csv) with scores for all links and for literature-validated links
- Summary statistics comparing mean IOKR score of all links vs validated links (reference values: 0.0105 vs 0.0364)

## How to apply

Set up the NPLinker working directory with nplinker.toml (root_dir, plus gnps/, antismash/, bigscape/ and strain_mappings.json in the fixed structure) and load all BGC, GCF, spectrum, molecular family and strain objects with npl.load_data(). Run IOKR scoring over candidate links between the structure-assignable BGCs/GCFs and spectra (e.g. the 2242 structure-assignable BGCs and 6246 MS2 spectra of the Crüsemann data set) by passing the scoring-method name to npl.get_links(). Extract per-link scores from the returned LinkGraph via link_graph.links and link_graph.get_link_data(), then compute summary statistics such as the mean score across all links versus the subset of literature-validated links (in the reference case, mean IOKR score 0.0105 for all links vs 0.0364 for validated links, Table 1/Fig 6). Because IOKR is a novel Input-Output Kernel Regression score, it is best interpreted alongside complementary scoring functions rather than in isolation.

## Related tools

- **NPLinker** (Framework in which the IOKR scoring method is invoked via npl.get_links() to compute and score candidate genomic–metabolomic links) — https://github.com/NPLinker/nplinker
- **nplinker** (Python package providing load_data(), get_links(), LinkGraph.links and LinkGraph.get_link_data() used to extract per-link IOKR scores) — https://github.com/NPLinker/nplinker
- **AntiSMASH** (Supplies the BGC data in the antismash/ directory that constitute one side of the scored links)
- **BigScape** (Clusters AntiSMASH BGCs into GCFs (run automatically by NPLinker if bigscape/ output is absent) used as link endpoints)
- **GNPS** (Source of the molecular networking data whose MS2 spectra form the metabolomic side of the scored links)

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs, "iokr"); data = link_graph.get_link_data()
```

## Evaluation signals

- Mean IOKR score over all links should be substantially lower than over literature-validated links (reference: 0.0105 vs 0.0364 on the Crüsemann data set)
- Number of structure-assignable BGCs and spectra scored matches the data set scale (e.g. 2242 BGCs and 6246 MS2 spectra)
- Per-link scores are retrievable from the returned LinkGraph via link_graph.links and link_graph.get_link_data() and export cleanly to iokr_scores.csv
- Score distribution shows enrichment of validated links at the upper end (cf. Fig 6), not a flat distribution
- All input objects (BGC, GCF, spectrum, molecular family, strain) were loaded successfully by npl.load_data() before scoring

## Limitations

- IOKR is a novel score whose interpretation benefits from being used together with complementary scoring functions; using multiple link-scoring functions together makes it easier to prioritise true links relative to others
- Results depend on correct working-directory structure (gnps/, antismash/, bigscape/, strain_mappings.json) and strain mappings; missing or malformed mappings will break link construction
- Benchmark values are tied to publicly available data sets with validated links (e.g. Crüsemann data set); other data sets lack a validated-link subset for equivalent verification

## Evidence

- [abstract] A novel score based on Input-Output Kernel Regression is introduced.: "introduce a novel score using an Input-Output Kernel Regression approach"
- [abstract] Using multiple scoring functions together improves prioritisation of true links.: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [methods] NPLinker links genomics and metabolomics data.: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
- [methods] Links are computed with npl.get_links() using a named scoring method.: "# compute the links for the first 3 GCFs using metcalf scoring method
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
