---
name: bottom-up-candidate-gene-output-parsing
description: Use when you have run ProGeM (or SNiPA) on sentinel mQTL variants and
  need to parse per-locus candidate gene calls, score sensitivity against a curated
  causal-gene reference, and benchmark false positives.
when_to_use_negative:
- You need top-down or cooccurring ('concurrent') candidate genes — parse 'OUTPUT_top_down_scores.txt'
  or the combined candidate strategy instead
- Your QTLs are trans-pQTLs, where ProGeM's positional top-down strategy is not directly
  applicable and a different approach is required
- No high-confidence reference causal gene set is available for your molecular QTL
  type, making sensitivity/FP scoring uninterpretable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Produces the bottom-up summary output that is parsed; supplied as a configurable
    R script
  repo: https://github.com/ds763/ProGeM
- name: SNiPA
  role: Comparison tool whose block annotation candidate gene calls are parsed and
    benchmarked against ProGeM
- name: R
  role: Language used to load, group and score the bottom-up summary rows into the
    benchmark table
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:351a5411cbf7952b6a380409794753e25501fa5cb85411bd7c2b4e0e46d864b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - SNiPA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bottom-up-candidate-gene-output-parsing

## Summary

Parse ProGeM's 'OUTPUT_bottom_up_summary.txt' to extract bottom-up candidate mediating genes per sentinel rsID, map them to a reference causal gene set, and compute sensitivity and false-positive counts (e.g. against the 227 curated mQTLs and tools like SNiPA).

## When to use

Use when you have run ProGeM (or SNiPA) on sentinel mQTL variants and need to parse per-locus candidate gene calls, score sensitivity against a curated causal-gene reference, and benchmark false positives.

## When NOT to use

- You need top-down or cooccurring ('concurrent') candidate genes — parse 'OUTPUT_top_down_scores.txt' or the combined candidate strategy instead
- Your QTLs are trans-pQTLs, where ProGeM's positional top-down strategy is not directly applicable and a different approach is required
- No high-confidence reference causal gene set is available for your molecular QTL type, making sensitivity/FP scoring uninterpretable

## Inputs

- ProGeM 'OUTPUT_bottom_up_summary.txt' (tab-separated, 12 columns: LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, nearest, second_nearest, third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead_IMPACT, proxy_IMPACT)
- Reference causal gene set for the 227 curated metabolite QTLs (Supplementary Table S1)
- SNiPA block annotation output (GRCh37, 1000 Genomes Phase 3 v5, European, r2=0.8) for the 218 sentinel rsIDs

## Outputs

- Per-sentinel-rsID list of bottom-up candidate mediating genes (ensembl_id, hgnc_symbol, gene_biotype)
- Sensitivity and false-positive counts per tool (SNiPA 201/218 = 92% with 1315 FPs; ProGeM bottom-up 209/218 = 96% with 827 FPs; combined ProGeM 83% with 207 FPs)
- Tab-separated benchmark table, e.g. snipa_vs_progem_bottomup_benchmark.tsv

## How to apply

Read 'OUTPUT_bottom_up_summary.txt' where column 1 holds the LEAD_rsID and columns 5-12 are booleans/IMPACT codes annotating each bottom-up candidate (nearest, second_nearest, third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead_IMPACT, proxy_IMPACT; IMPACT 0=MODIFIER/LOW, 1=MODERATE, 2=HIGH). Group rows by sentinel rsID to collect ProGeM's candidate mediating genes, then match them against the reference causal genes for the 218 recognizable sentinels of the 227 curated mQTLs. Compute sensitivity as the fraction of reference causal genes recovered (ProGeM bottom-up: 209/218 = 96%; SNiPA: 201/218 = 92%) and count candidate genes outside the reference set as false positives (ProGeM: 827; SNiPA: 1315). Use R (ProGeM is delivered as R scripts using data.table/GenomicRanges) to assemble the metrics into a tab-separated comparison table such as snipa_vs_progem_bottomup_benchmark.tsv.

## Related tools

- **ProGeM** (Produces the bottom-up summary output that is parsed; supplied as a configurable R script) — https://github.com/ds763/ProGeM
- **SNiPA** (Comparison tool whose block annotation candidate gene calls are parsed and benchmarked against ProGeM)
- **R** (Language used to load, group and score the bottom-up summary rows into the benchmark table) — https://github.com/ds763/ProGeM

## Examples

```
snipa_fp <- sum(!candidates_snipa$gene %in% ref_causal$gene); snipa_sens <- sum(candidates_snipa$gene %in% ref_causal$gene) / 218  # expect 201/218, 1315 FPs; write.table(bench, 'snipa_vs_progem_bottomup_benchmark.tsv', sep='\t')
```

## Evaluation signals

- SNiPA sensitivity reproduces 201/218 (= 92%) with exactly 1315 false positives over the 218 recognizable sentinels
- ProGeM bottom-up sensitivity reproduces 209/218 (= 96%) with exactly 827 false positives
- Combined ProGeM strategy yields 83% sensitivity with 207 false positives, matching the expected benchmark integers exactly
- Each parsed row retains a valid LEAD_rsID in column 1 and boolean/IMPACT values in columns 5-12 consistent with the README schema
- Output file snipa_vs_progem_bottomup_benchmark.tsv is a valid tab-separated table containing all tool-by-metric rows

## Limitations

- Reference causal gene assignments at the 227 mQTLs may be biased, e.g. toward prioritization of the nearest genes
- ProGeM and the curated mQTL dataset are not fully independent (KEGG was used both by ProGeM and to guide the expert literature review), so mQTL sensitivity may be inflated
- Only 218 of the 227 sentinel variants were recognizable by SNiPA, capping the comparable sensitivity denominators

## Evidence

- [readme] Each row provides annotation of a "bottom-up" candidate mediating gene, the corresponding sentinel rsID is indicated in column 1: "Each row provides annotation of a *"bottom-up"* candidate mediating gene, the corresponding sentinel rsID is indicated in **column 1**"
- [readme] columns 11-12 IMPACT coding: "**columns 11-12** are nominal variables referring to VEP IMPACT annotation with 0=MODIFIER or LOW IMPACT, 1=MODERATE IMPACT, 2=HIGH IMPACT."
- [other] SNiPA vs ProGeM sensitivity findings: [evidence span withheld — anchor status: not_found]
- [abstract] reference set of 227 mQTLs: "a reference set of 227 previously reported and extensively curated metabolite QTLs"
- [methods] proxy r2 filtering default: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
