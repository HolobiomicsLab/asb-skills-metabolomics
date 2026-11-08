---
name: pre-ranked-pvalue-lipid-ranking
description: Use when a differential-expression results table from a two-group lipidomics
  comparison is available and the goal is to test whether a priori-defined lipid sets
  (by class, chain length, unsaturation, hydroxyl groups, or fatty acid composition)
  are enriched, rather than testing individual lipids.
when_to_use_negative:
- Data is multi-group rather than two-group differential-expression output — the LSEA
  demonstration and differential-expression ranking are framed for case-vs-control
  designs
- Goal is Over Representation Analysis (ORA) on a pre-selected significant lipid list
  rather than a pre-ranked full-lipid ranking by p-value
- Analysis goal is pathway/network-level enrichment (Network/GATOM workflow) rather
  than lipid-characteristic set enrichment
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: LipidSigR
  role: Provides enrichment_lsea (pre-ranked LSEA by p-value) and plot_enrichment_lsea
    (bar plot and enrichment plot for a selected lipid class) within the Enrichment
    workflow
  repo: https://github.com/BioinfOMICS/LipidSigR
- name: R
  role: Runtime environment (R 4.3.0 or later) in which the LSEA ranking and plotting
    functions are executed on the differential-expression table
  repo: https://github.com/BioinfOMICS/LipidSigR
- name: dplyr
  role: Data manipulation of the differential-expression output table before ranking
    and subsetting LSEA results by lipid class
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf047
    title: 'LipidSigR: a R-based solution for integrated lipidomics data analysis
      and visualization'
  source_content_hash: sha256:743aec3bc8e45bcbfce19ff7b38bbe55e9992a663ac352bf980d122a93c2fdb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfOMICS/LipidSigR
  tools:
  - LipidSigR
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Pre-ranked p-value lipid ranking

## Summary

Ranks lipid species by their differential-expression p-values and applies Lipid Set Enrichment Analysis (LSEA) via LipidSigR's enrichment_lsea to identify which lipid-characteristic sets (e.g., TAG, PC O-, Cer) are enriched among up- or down-regulated species, with visualization via plot_enrichment_lsea.

## When to use

Use when a differential-expression results table from a two-group lipidomics comparison is available and the goal is to test whether a priori-defined lipid sets (by class, chain length, unsaturation, hydroxyl groups, or fatty acid composition) are enriched, rather than testing individual lipids one at a time.

## When NOT to use

- Data is multi-group rather than two-group differential-expression output — the LSEA demonstration and differential-expression ranking are framed for case-vs-control designs
- Goal is Over Representation Analysis (ORA) on a pre-selected significant lipid list rather than a pre-ranked full-lipid ranking by p-value
- Analysis goal is pathway/network-level enrichment (Network/GATOM workflow) rather than lipid-characteristic set enrichment

## Inputs

- Differential-expression results table from LipidSigR's de_analysis step (lipid species with p-values and fold changes)
- Input SummarizedExperiment object constructed with LipidSigR tool functions (upstream prerequisite)

## Outputs

- LSEA results table of enriched lipid sets per lipid characteristic (e.g., lipid class) with enrichment statistics
- lsea_bar_plot_Cer.png — bar plot of enriched lipid sets for the selected class
- lsea_enrichment_plot_Cer.png — enrichment (running enrichment score) plot for the selected class

## How to apply

Load the differential-expression output table produced by LipidSigR's upstream Differential Expression workflow into R. Call LipidSigR's enrichment_lsea function with rank_by='pval' so lipids are ranked by differential-expression p-values in a pre-ranked manner (GSEA/fgsea-style, using the fgsea dependency), yielding per-lipid-class enrichment statistics. Inspect the LSEA results table and select the lipid class or classes of interest (e.g., ceramide/Cer) for visualization. Use plot_enrichment_lsea to generate both the bar plot of enriched lipid sets and the running-enrichment plot for the chosen class, exporting each as PNG. Rationale: LSEA evaluates whether an a priori-defined set of lipids shows statistically significant, concordant differences between biological states without requiring a user-supplied significance cutoff on individual lipids.

## Related tools

- **LipidSigR** (Provides enrichment_lsea (pre-ranked LSEA by p-value) and plot_enrichment_lsea (bar plot and enrichment plot for a selected lipid class) within the Enrichment workflow) — https://github.com/BioinfOMICS/LipidSigR
- **R** (Runtime environment (R 4.3.0 or later) in which the LSEA ranking and plotting functions are executed on the differential-expression table) — https://github.com/BioinfOMICS/LipidSigR
- **dplyr** (Data manipulation of the differential-expression output table before ranking and subsetting LSEA results by lipid class)

## Examples

```
lsea_result <- enrichment_lsea(de_result, rank_by = "pval"); plot_enrichment_lsea(lsea_result, class = "Cer", saveTo = "lsea_enrichment_plot_Cer.png")
```

## Evaluation signals

- Both exported PNG files (lsea_bar_plot_Cer.png, lsea_enrichment_plot_Cer.png) exist and are non-empty
- The plotted enrichment results correspond to the Cer-class entries in the enrichment_lsea output table
- Ranking uses rank_by='pval', so the ordering of lipids in the pre-ranked vector matches ascending p-values from the differential-expression table
- Enriched sets are lipid-characteristic categories (e.g., TAG for up-regulated, PC O- for down-regulated species) consistent with the direction of regulation in the differential-expression results
- The function runs under the documented environment (R >= 4.3.0 with LipidSigR and its fgsea dependency installed)

## Limitations

- LipidSigR requires R 4.3.0 or later, and on Windows is supported only on Windows 10 or later
- Only two-group data can conduct the Network and Machine learning workflows, so downstream integration of LSEA results with those workflows is unavailable for multi-group data
- LSEA results depend on the quality and completeness of the upstream differential-expression table; no individual-lipid significance cutoff is applied before ranking

## Evidence

- [readme] Enrichment: ORA and LSEA approaches: "The enrichment workflow provides two main approaches: 'Over Representation Analysis (ORA)' and 'Lipid Set Enrichment Analysis (LSEA)' to illustrates significant lipid species enriched in the"
- [readme] Concordant differences between biological states: [evidence span withheld — anchor status: external_doc]
- [intro] R version requirement: "You need to have R 4.3.0 or a later version installed for running LipidSigR."
- [intro] Lipid characteristics analyzed: "Each section offers unique aspects to analyzing lipidome profiling data based on various characteristics, including lipid class, chain length, unsaturation, hydroxyl groups, and fatty acid"
- [full_text] Workflow demonstration with enrichment_lsea rank_by='pval': [evidence span withheld — anchor status: not_found]
