---
name: fdr-multiple-testing-correction
description: Use when interpreting over-representation pathway enrichment results
  from multiple metabolite lists, where p < 0.05 hits exist but only FDR-adjusted
  values below 0.05 should count as significant pathways.
when_to_use_negative:
- When a single pathway or hypothesis is being tested with no multiple-testing correction
  needed (a lone raw p-value suffices).
- When the input is a raw metabolite feature list that still needs harmonization/merging
  (e.g., unmerged datasets requiring MMS with InChIKey translation) before any enrichment
  or FDR assessment.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst 5.0
  role: Runs the over-representation pathway enrichment against SMPDB and reports
    p-value and FDR for each pathway in both the MMS-merged and unmerged conditions.
- name: SMPDB
  role: Small Molecule Pathway Database supplying the pathway universe over which
    enrichment p-values and FDR are computed.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:1c13ab8407152fa2bd0b8a5f6ec98a1f88dc7ab074c352f6332804447d3a260d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 5.0
  - SMPDB
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fdr-multiple-testing-correction

## Summary

Apply false discovery rate (FDR) correction to pathway enrichment p-values when comparing metabolite sets (e.g., MMS-merged vs. unmerged urinary asthma metabolites) so that significance calls (FDR < 0.05) in MetaboAnalyst 5.0/SMPDB runs reflect multiple-testing burden rather than raw p < 0.05 hits.

## When to use

Use when interpreting over-representation pathway enrichment results from multiple metabolite lists, where p < 0.05 hits exist but only FDR-adjusted values below 0.05 should count as significant pathways.

## When NOT to use

- When a single pathway or hypothesis is being tested with no multiple-testing correction needed (a lone raw p-value suffices).
- When the input is a raw metabolite feature list that still needs harmonization/merging (e.g., unmerged datasets requiring MMS with InChIKey translation) before any enrichment or FDR assessment.

## Inputs

- Pathway enrichment result tables from MetaboAnalyst 5.0 (SMPDB) containing per-pathway p-values and FDR values for each metabolite input list

## Outputs

- Side-by-side comparison table of p-values and FDR per pathway across runs, saved as pathway_enrichment_comparison.csv
- Set of significant pathways defined as FDR < 0.05 (e.g., tryptophan metabolism and beta-alanine metabolism)

## How to apply

Run over-representation analysis in MetaboAnalyst 5.0 against the Small Molecule Pathway Database (SMPDB) for each compound list (e.g., the MMS-merged urinary asthma set filtered to compounds found in at least two information sources, and the same list without MMS harmonization). For every pathway, extract both the raw p-value and the FDR; treat only pathways with FDR < 0.05 as significant, while pathways with p < 0.05 alone are 'relevant'. Build a side-by-side comparison table of p-value and FDR per pathway across the merged and unmerged runs — in the reference case, this distinction is decisive: seven pathways were relevant (p < 0.05) and two significant (FDR < 0.05; tryptophan and beta-alanine metabolism) with MMS, whereas without MMS three were relevant but none reached significance. Report significance strictly at the FDR level when comparing enrichment outcomes across strategies.

## Related tools

- **MetaboAnalyst 5.0** (Runs the over-representation pathway enrichment against SMPDB and reports p-value and FDR for each pathway in both the MMS-merged and unmerged conditions.)
- **SMPDB** (Small Molecule Pathway Database supplying the pathway universe over which enrichment p-values and FDR are computed.)

## Evaluation signals

- The MMS run reproduces exactly two significant pathways at FDR < 0.05: tryptophan metabolism and beta-alanine metabolism.
- The no-MMS run shows zero significant pathways (FDR < 0.05) even though three pathways are relevant at p < 0.05.
- Every pathway row in the comparison table carries both a p-value and an FDR from the MetaboAnalyst 5.0 / SMPDB output.
- Relevant vs. significant classification is consistent: significance implies FDR < 0.05, and 'relevance' alone (p < 0.05) never counts as significant.
- The merged/unmerged contrast is saved as pathway_enrichment_comparison.csv with side-by-side p-values and FDR values.

## Limitations

- Raw p < 0.05 ('relevant') hits can overstate the evidence: without MMS merging, three pathways were relevant but none achieved FDR significance, so relying on p-values alone changes the biological conclusion.
- Significance depends on the input compound list — applying MMS harmonization (391 merged metabolites filtered to compounds in at least two sources vs. 547 unmerged entries) alters which pathways pass FDR < 0.05.
- FDR thresholds here are tied to over-representation analysis in MetaboAnalyst 5.0/SMPDB; results may not transfer to other pathway databases or enrichment methods.

## Evidence

- [abstract] MMS applied to urinary asthma metabolites identified significant pathways that were hidden when no dataset merging strategy was followed: "MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed"
- [supplementary] Reproduce the enrichment analysis with vs without MMS: [evidence span withheld — anchor status: not_found]
- [supplementary] Significance threshold definition: [evidence span withheld — anchor status: not_found]
