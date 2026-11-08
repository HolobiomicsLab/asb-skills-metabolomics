---
name: merged-vs-unmerged-enrichment-comparison
description: Use when you hold two versions of a metabolite list — one harmonized/merged
  (e.g., via MMS InChIKey merging) and one unmerged — and need to quantify how harmonization
  changes which pathways reach significance (FDR < 0.05).
when_to_use_negative:
- You have only a single metabolite list with no merged/unmerged counterpart — there
  is no contrast to draw; run a single over-representation enrichment instead.
- The task is performing the MMS merging itself (InChIKey translation, attribute retrieval,
  three-step curation) rather than measuring its downstream effect on enrichment.
- The enrichment uses a method or pathway library other than over-representation against
  SMPDB in MetaboAnalyst 5.0 — the 2-vs-0 significance benchmark does not transfer.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetaboAnalyst 5.0
  role: Runs the over-representation pathway enrichment on both the MMS-merged and
    unmerged compound lists, returning per-pathway p-value and FDR
- name: SMPDB
  role: Pathway reference library (Small Molecule Pathway Database) against which
    over-representation is tested in both runs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:a465722eea4bddd708ebbe5c1fc9bbc492fa0439f665b6b1a238ea76e9060381
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 5.0
  - SMPDB
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# merged-vs-unmerged-enrichment-comparison

## Summary

Benchmarks how metabolite identifier harmonization changes pathway enrichment results by rerunning the identical MetaboAnalyst 5.0 over-representation analysis (SMPDB pathways) on an MMS-merged urinary asthma metabolite list (391 metabolites) versus the unmerged 547-entry list, and comparing per-pathway p-values and FDR side by side.

## When to use

Use when you hold two versions of a metabolite list — one harmonized/merged (e.g., via MMS InChIKey merging) and one unmerged — and need to quantify how harmonization changes which pathways reach significance (FDR < 0.05).

## When NOT to use

- You have only a single metabolite list with no merged/unmerged counterpart — there is no contrast to draw; run a single over-representation enrichment instead.
- The task is performing the MMS merging itself (InChIKey translation, attribute retrieval, three-step curation) rather than measuring its downstream effect on enrichment.
- The enrichment uses a method or pathway library other than over-representation against SMPDB in MetaboAnalyst 5.0 — the 2-vs-0 significance benchmark does not transfer.

## Inputs

- MMS-merged urinary asthma metabolite dataset (391 metabolites, Supplementary Table S2), filtered to compounds found in at least two information sources
- Unmerged urinary asthma metabolite dataset (547 entries, same compounds without MMS harmonization), filtered to compounds found in at least two information sources
- SMPDB (Small Molecule Pathway Database) pathway library accessed via MetaboAnalyst 5.0's over-representation module

## Outputs

- pathway_enrichment_comparison.csv: side-by-side table of p-value and FDR for every pathway from the merged and unmerged runs
- Per-run counts of relevant (p < 0.05) and significant (FDR < 0.05) pathways (7/2 with MMS vs 3/0 without)

## How to apply

Build two compound lists from the urinary asthma case study: the MMS-merged set (391 metabolites, Supplementary Table S2) and the unmerged 547-entry set, filtering both to compounds found in at least two information sources so harmonization is the intended difference. Run over-representation pathway enrichment in MetaboAnalyst 5.0 against the Small Molecule Pathway Database (SMPDB) on each list, keeping all parameters identical between the two runs. Extract the p-value and false discovery rate (FDR) for every pathway from both runs and assemble a single side-by-side comparison table, saved as pathway_enrichment_comparison.csv. The rationale is that MMS consolidates duplicated identifiers (synonym checking, conjugate base/acid curation on InChIKeys) so hits concentrate in fewer pathways, rescuing significance that unmerged synonym-laden lists dilute. Judge success by the paper's outcome: the MMS run yields seven relevant pathways (p < 0.05) and exactly two significant ones (FDR < 0.05, tryptophan and beta-alanine metabolism), while the unmerged run yields three relevant (p < 0.05) and none significant.

## Related tools

- **MetaboAnalyst 5.0** (Runs the over-representation pathway enrichment on both the MMS-merged and unmerged compound lists, returning per-pathway p-value and FDR)
- **SMPDB** (Pathway reference library (Small Molecule Pathway Database) against which over-representation is tested in both runs)

## Examples

```
python -c "import pandas as pd; cmp=mms.merge(unmerged,on='pathway',suffixes=('_mms','_nomms')); cmp.to_csv('pathway_enrichment_comparison.csv')"
```

## Evaluation signals

- MMS run reproduces exactly two significant pathways at FDR < 0.05: tryptophan metabolism and beta-alanine metabolism.
- No-MMS run shows zero pathways reaching FDR < 0.05, with three relevant hits at p < 0.05.
- MMS run shows seven relevant pathways at p < 0.05.
- Both runs use identical parameters: MetaboAnalyst 5.0 over-representation against SMPDB, with the same compounds-in-≥2-sources filter applied to each list (391 merged vs 547 unmerged entries before filtering).
- Comparison table contains p-value and FDR columns for every pathway from both runs and is saved as pathway_enrichment_comparison.csv.

## Limitations

- The benchmark (2 significant pathways with MMS vs 0 without) is specific to the urinary asthma metabolite case study and SMPDB over-representation; it need not transfer to other datasets or pathway libraries.
- Filtering to compounds found in at least two information sources changes the analyzed list size (391 merged metabolites vs 547 unmerged entries), so the contrast reflects harmonization plus list composition.
- Without a merging strategy, significant pathways remain hidden: the unmerged list yields only three relevant pathways (p < 0.05) and none at FDR < 0.05, so conclusions depend on the upstream harmonization step.

## Evidence

- [abstract] MMS reveals significant pathways hidden without dataset merging: "MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed"
- [methods] Enrichment method and pathway database: [evidence span withheld — anchor status: not_found]
- [results] Merged vs unmerged significance counts: [evidence span withheld — anchor status: not_found]
