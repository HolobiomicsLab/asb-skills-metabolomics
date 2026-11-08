---
name: cross-source-metabolite-frequency-filtering
description: Use when you have a merged/harmonized metabolite table from multiple
  information sources (e.g. an MMS InChIKey-merged list) and need to reduce false
  positives before pathway enrichment by keeping only compounds supported by ≥2 sources.
when_to_use_negative:
- You have a single-source list (no frequency information) — no filtering is possible.
- You want maximum sensitivity with no confidence filter (the ≥2-source cutoff is
  a confidence requirement).
- The target downstream tool needs raw counts rather than a filtered compound list.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst 5.0
  role: Pathway over-representation analysis tool that consumes the filtered metabolite
    list (with SMPDB pathways) to compute p-values and FDRs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:f593204dc37abd94ab50af66ce09a66833c6d144d0e24c8d6f23282ef3d3cf9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 5.0
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-source-metabolite-frequency-filtering

## Summary

This skill applies a minimum-source-frequency filter to a harmonized metabolite list (e.g. MMS output) so that only compounds corroborated by at least two information sources are carried into downstream analyses such as pathway enrichment, improving confidence in the merged dataset.

## When to use

Use when you have a merged/harmonized metabolite table from multiple information sources (e.g. an MMS InChIKey-merged list) and need to reduce false positives before pathway enrichment by keeping only compounds supported by ≥2 sources.

## When NOT to use

- You have a single-source list (no frequency information) — no filtering is possible.
- You want maximum sensitivity with no confidence filter (the ≥2-source cutoff is a confidence requirement).
- The target downstream tool needs raw counts rather than a filtered compound list.

## Inputs

- MMS-merged metabolite table (e.g. 391 compounds, Supplementary Table S2) with per-compound source counts
- Unmerged metabolite list (547 entries) with per-compound source counts
- Mapping of metabolite identifiers to pathway databases (SMPDB) in MetaboAnalyst 5.0

## Outputs

- Filtered metabolite list (compounds found in ≥2 information sources), suitable for over-representation analysis
- Optional: side-by-side enrichment comparison (p-value, FDR) showing two significant pathways (tryptophan, beta-alanine metabolism) with MMS vs none without

## How to apply

Take the MMS-merged table (e.g. 391 urinary asthma metabolites from Supplementary Table S2) which already carries source counts per compound, and filter to compounds appearing in at least two of the information sources (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Metabolomics Workbench, etc.). The article used this ≥2-source threshold on both the merged (391) and unmerged (547-entry) versions before running MetaboAnalyst 5.0 / SMPDB over-representation analysis, so the same threshold should be used if comparing merged vs unmerged enrichment. Output the filtered compound list (names plus identifiers) for input to the enrichment tool; validate by counting remaining compounds and confirming the MMS-filtered set still yields the reported two significant pathways (tryptophan and beta-alanine metabolism, FDR < 0.05) while the unmerged filtered set yields none.

## Related tools

- **MetaboAnalyst 5.0** (Pathway over-representation analysis tool that consumes the filtered metabolite list (with SMPDB pathways) to compute p-values and FDRs)

## Examples

```
python filter_by_source_count.py --input mms_merged_391.csv --min_sources 2 --output filtered_compounds.csv
```

## Evaluation signals

- Count of retained compounds must be less than the pre-filter total (e.g. <391 for the MMS set, <547 for the unmerged set).
- Every retained compound must have a source count ≥2 (no single-source compounds remain).
- Running the filtered MMS list through MetaboAnalyst 5.0 / SMPDB must reproduce exactly two significant pathways at FDR < 0.05 (tryptophan and beta-alanine metabolism).
- Running the filtered unmerged list through the same pipeline must yield zero significant pathways despite any p < 0.05 hits.
- A side-by-side comparison table (pathway_enrichment_comparison.csv) should show the MMS-filtered set outperforming the unmerged-filtered set in FDR significance.

## Limitations

- The ≥2-source threshold is dataset-specific; the article does not state whether it is universally optimal.
- Filtering removes potentially real single-source metabolites, which could reduce pathway coverage in some analyses.
- Downstream enrichment results depend on the completeness of the SMPDB/MetaboAnalyst mapping for the retained identifiers.

## Evidence

- [methods] 391 metabolites, Supplementary Table S2: [evidence span withheld — anchor status: not_found]
- [methods] 547 entries: [evidence span withheld — anchor status: not_found]
- [results] two significant pathways (FDR < 0.05): [evidence span withheld — anchor status: not_found]
- [abstract] MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed: "MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed"
