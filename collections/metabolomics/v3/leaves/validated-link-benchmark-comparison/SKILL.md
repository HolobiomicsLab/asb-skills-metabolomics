---
name: validated-link-benchmark-comparison
description: Use when a microbial genome-metabolome linking score has been computed
  over candidate BGC/spectrum links and you need to quantify whether the score's upper
  tail is enriched for literature-validated links, as in Table 1 / Fig 6.
when_to_use_negative:
- No literature-validated link list exists for the data set, so no benchmark subset
  can be defined
- The scoring method being evaluated is not one of the link-scoring functions available
  via npl.get_links()
- The data set is not assembled in the NPLinker fixed directory structure (gnps/,
  antismash/, bigscape/, strain_mappings.json), since load_data() expects it
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: NPLinker
  role: Framework that loads BGC, GCF, spectrum, molecular family and strain objects
    and computes candidate links with IOKR scoring via npl.get_links()
  repo: https://github.com/NPLinker/nplinker
- name: nplinker
  role: Python package installed with pip (~300MB) providing load_data(), get_links(),
    LinkGraph.links and get_link_data() used to extract per-link scores
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Runtime for the benchmark script (Python >= 3.11); computes mean scores and
    writes iokr_scores.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:a083638ef06cd989df1ffa31fdc63cc233e1f82fa6849c0baaca12422baba6f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - NPLinker
  - Python
  - nplinker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# validated-link-benchmark-comparison

## Summary

Benchmarks a computed link-scoring function (e.g. IOKR) against literature-validated BGC-spectrum links by comparing score distributions over all candidate links versus the validated subset, replicating the Crüsemann data set evaluation in the NPLinker framework.

## When to use

Use when a microbial genome-metabolome linking score has been computed over candidate BGC/spectrum links and you need to quantify whether the score's upper tail is enriched for literature-validated links, as in Table 1 / Fig 6.

## When NOT to use

- No literature-validated link list exists for the data set, so no benchmark subset can be defined
- The scoring method being evaluated is not one of the link-scoring functions available via npl.get_links()
- The data set is not assembled in the NPLinker fixed directory structure (gnps/, antismash/, bigscape/, strain_mappings.json), since load_data() expects it

## Inputs

- NPLinker working directory with nplinker.toml (root_dir) containing gnps/, antismash/, bigscape/ subdirectories and strain_mappings.json
- Crüsemann data set: 2242 structure-assignable BGCs/GCFs and 6246 MS2 spectra
- List of literature-validated BGC-spectrum links used as the benchmark subset

## Outputs

- Per-link IOKR scores exported to iokr_scores.csv
- Mean IOKR score over all links (expected ~0.0105)
- Mean IOKR score over validated links (expected ~0.0364)
- Comparison against reported Table 1 / Fig 6 values

## How to apply

Set up the NPLinker working directory for the Crüsemann data set (nplinker.toml with root_dir; gnps/, antismash/, bigscape/ directories and strain_mappings.json) and load all BGC, GCF, spectrum, molecular family and strain objects with npl.load_data(). Run the IOKR (Input-Output Kernel Regression) scoring over candidate links between the 2242 structure-assignable BGCs/GCFs and the 6246 MS2 spectra using npl.get_links(). Extract per-link scores from the returned LinkGraph via link_graph.links and link_graph.get_link_data(). Compute the mean score across all links and across the literature-validated subset, then compare against the reference values (all links mean 0.0105; validated links mean 0.0364, Table 1 / Fig 6), which indicates the upper end of the score distribution is enriched for validated links. Write the per-link scores to iokr_scores.csv for traceability.

## Related tools

- **NPLinker** (Framework that loads BGC, GCF, spectrum, molecular family and strain objects and computes candidate links with IOKR scoring via npl.get_links()) — https://github.com/NPLinker/nplinker
- **nplinker** (Python package installed with pip (~300MB) providing load_data(), get_links(), LinkGraph.links and get_link_data() used to extract per-link scores) — https://github.com/NPLinker/nplinker
- **Python** (Runtime for the benchmark script (Python >= 3.11); computes mean scores and writes iokr_scores.csv)

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs, "iokr"); scores = link_graph.get_link_data()
```

## Evaluation signals

- Mean IOKR score over all links in the Crüsemann data set reproduces the reported value of 0.0105
- Mean IOKR score over the literature-validated link subset reproduces the reported value of 0.0364 (validated mean ~3.5x the all-links mean)
- Link count matches the expected candidate space of 2242 structure-assignable BGCs/GCFs against 6246 MS2 spectra
- link_graph.get_link_data() returns per-link score records for every link in LinkGraph.links with no missing scores
- iokr_scores.csv is written and contains both the full-link set and a distinguishable validated-subset column/flag

## Limitations

- Benchmark values are specific to the Crüsemann data set; other data sets will have different score distributions and cannot be checked against 0.0105 / 0.0364
- Validation relies on literature-validated links, which are only available for publicly available data sets that include them
- Requires the MIBiG metadata directory, which is automatically created and downloaded by NPLinker, so an offline environment may fail

## Evidence

- [abstract] Results verified on data with validated links: "Results are verified using publicly available data sets that include validated links."
- [methods] IOKR scoring link computation: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [methods] Data loading step: "# load data
npl.load_data()"
- [methods] MIBiG dependency note: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"
