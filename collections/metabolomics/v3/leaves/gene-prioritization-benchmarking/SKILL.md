---
name: gene-prioritization-benchmarking
description: Use when a gene prioritization method must be quantitatively validated
  against known causative genes — via permutation tests, percentile-of-causative-gene
  metrics, or ablations of the underlying interaction network.
when_to_use_negative:
- There are no known causative genes or validated diagnoses for the cohort, so percentile-of-causative-gene
  metrics cannot be computed
- Candidate gene lists are already so small (e.g., trio WES with parental data) that
  top-20th-percentile ranking is trivially uninformative — the authors note trio WES
  shortens candidate lists versus their singleton exomes
- The goal is discovery rather than validation — benchmarking consumes known disease-gene
  assignments (e.g., OMIM) and cannot be run on undiagnosed patients
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0605
tools:
- name: metPropagate
  role: Supplies the label-propagation candidate gene scores (propagated ME scores
    over the STRING functional linkage network) that are being benchmarked
  repo: github.com/emmagraham/metPropagate
- name: Exomiser
  role: Provides the competing phenotype-based (hiPHIVE) ranking and the variant-filtering
    pipeline that generates the candidate gene lists being scored
- name: STRING
  role: Source of the protein functional linkage network; full vs physical-only ('binding'-edge)
    versions are ablated to test the contribution of functional edges
- name: yamaguchiyuto/label_propagation
  role: scikit-compatible label propagation implementation (LGC, alpha=0.99) adapted
    by metPropagate to produce per-gene propagated scores for benchmarking
  repo: https://github.com/yamaguchiyuto/label_propagation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41525-020-0132-5
    title: 'metPropagate: network-guided propagation of metabolomic information for
      prioritization of metabolic disease genes'
  source_content_hash: sha256:4e2a7c2e5d45471c2acf2ad9c878a6d5959f0875a0ec5334002dcf299f6efd06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Exomiser
  - STRING
  - metPropagate
  - yamaguchiyuto/label_propagation
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-prioritization-benchmarking

## Summary

Benchmarks a gene prioritization method (e.g., metPropagate label-propagation scores) by measuring how often causative genes fall in the top 20th percentile of candidate lists, using permutation controls, head-to-head tool comparisons (Exomiser), and network ablations (full vs physical-only STRING).

## When to use

Use when a gene prioritization method must be quantitatively validated against known causative genes — via permutation tests, percentile-of-causative-gene metrics, or ablations of the underlying interaction network.

## When NOT to use

- There are no known causative genes or validated diagnoses for the cohort, so percentile-of-causative-gene metrics cannot be computed
- Candidate gene lists are already so small (e.g., trio WES with parental data) that top-20th-percentile ranking is trivially uninformative — the authors note trio WES shortens candidate lists versus their singleton exomes
- The goal is discovery rather than validation — benchmarking consumes known disease-gene assignments (e.g., OMIM) and cannot be run on undiagnosed patients

## Inputs

- Per-patient candidate gene lists from Exomiser variant filtering (281–609 genes per TIDEX patient)
- Known causative gene assignments for the 250 evaluated causative genes (Miller cohort, 107 IEM patients)
- Post-propagation node scores from metPropagate LPA_output (Score.ID / Original.Score.ID per gene)
- STRING v11 network edge tables — full functional and physical-only ('binding' edges) subsets
- Per-gene ME scores (scaled 0–1 DME enrichment p-value × |max metabolite z-score|)

## Outputs

- Fraction of patients with at least one causative gene in the top 20th percentile (e.g., 33/42 = 79% for no-ME-score patients)
- Permutation percentile distribution per patient from 1000 random 300-gene candidate lists
- Per-gene rank comparison table full-STRING vs physical-only-STRING (string_network_comparison.csv) plus summary fraction (~90%)
- Head-to-head win counts vs Exomiser (6/11) and weighted-combination comparison results

## How to apply

For each patient, generate 1000 permutations of random candidate gene lists (300 genes each) that include the causative gene, and compute where the causative gene's propagated score falls to derive a percentile. Report the fraction of patients for whom at least one causative gene lands in the top 20th percentile of the candidate list (candidate lists from Exomiser filtering contain 281–609 genes, so the top 20th percentile is ~55–115 genes). Benchmark against a competing prioritizer by comparing rank positions — e.g., count patients where metPropagate ranks the causative gene higher than Exomiser's hiPHIVE phenotype-based ranking — and test a weighted combination of the two scores for improved prioritization. For network ablation, run identical label propagation (lambda = 0.99, as in Zhou et al.) on the complete STRING network versus a physical-interactions-only subset (edges retaining the 'binding' signifier in the protein.links.full type column), then for each causative gene record its rank under both networks and compute the fraction ranked higher on the full network. Validate against the paper's reported values: 91–92% of Miller patients prioritized in the top 20th percentile, 9/11 TIDEX patients, 6/11 wins over Exomiser, and ~90% of causative genes ranked higher on the full functional network.

## Related tools

- **metPropagate** (Supplies the label-propagation candidate gene scores (propagated ME scores over the STRING functional linkage network) that are being benchmarked) — github.com/emmagraham/metPropagate
- **Exomiser** (Provides the competing phenotype-based (hiPHIVE) ranking and the variant-filtering pipeline that generates the candidate gene lists being scored)
- **STRING** (Source of the protein functional linkage network; full vs physical-only ('binding'-edge) versions are ablated to test the contribution of functional edges)
- **yamaguchiyuto/label_propagation** (scikit-compatible label propagation implementation (LGC, alpha=0.99) adapted by metPropagate to produce per-gene propagated scores for benchmarking) — https://github.com/yamaguchiyuto/label_propagation

## Examples

```
python main.py lgc -g string_network.edgelist -l sampleA_label -o LPA_output/sampleA --alpha 0.99
```

## Evaluation signals

- Reproduced fraction of causative genes ranked higher on the full STRING network should be approximately 90%, matching the paper's reported comparison
- Among patients whose causative genes lacked an ME score, one causative gene should appear in the top 20th percentile for ~79% of patients (33/42)
- Top 20th percentile of a 281–609 gene candidate list corresponds to 55–115 genes — check the percentile cutoff scales with list length
- Permutation test: causative gene's rank within 1000 random 300-gene candidate lists should significantly exceed chance
- Several causative genes should appear as missing (absent nodes) from the physical-only STRING network, confirming the ablation removed them

## Limitations

- Label propagation lambda = 0.99 was taken from Zhou et al. and not optimized, so benchmark numbers reflect untuned hyperparameters
- Analyses used singleton exomes rather than trio WES; the authors note trio parental data would shorten candidate lists and change percentile benchmarks
- Metabolomic systems cannot measure all metabolites (e.g., large glycosaminoglycans for the IDS patient), so diseases with unmeasured biomarkers are underrepresented in the benchmark
- Common IEM biomarkers such as acyl glycines and acyl carnitines are not annotated to IEM-associated genes in HMDB, limiting the ME-score baseline being benchmarked
- No STRING software version was reported, complicating exact reproduction of the network ablation

## Evidence

- [abstract] metPropagate was able to prioritize at least one causative gene in the top 20th percentile of candidate genes for 92% of patients with known IEMs.: "metPropagate was able to prioritize at least one causative gene in the top 20th percentile of candidate genes for 92% of patients with known IEMs."
- [results] 1000 permutations of random candidate gene lists that included the causative gene were generated: "1000 permutations of random candidate gene lists that included the causative gene were generated"
- [discussion] candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes: "candidate gene lists for a single patient included between 281 and 609 genes, meaning that the top 20th percentile included 55–115 genes"
- [other] Download the homo sapiens STRING v11 network from stringdb.org and subset it to physical interactions only by retaining edges with the 'binding' signifier in the type column of the protein.links.full file.: [evidence span withheld — anchor status: not_found]
- [readme] File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID".: "File contains final propagated scores for each candidate gene as "Score.ID" and the original scores for each gene as "Original.Score.ID"."
