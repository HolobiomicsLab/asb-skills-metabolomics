---
name: ccs-literature-value-collation
description: Use when benchmarking in-silico CCS predictions or a new instrument's
  TWIMS/DTIM measurements against literature CCS database values and you need a percent-deviation
  table keyed by metabolite and adduct state.
when_to_use_negative:
- The metabolites/adduct states of interest have no literature CCS database values
  available, so no percent-deviation collation is possible
- The input is raw instrument data (e.g., drift time or arrival-time distributions)
  rather than already-derived CCS values
- The analysis targets IM-MS data-collection or standardization procedures rather
  than value-level comparison against literature CCS
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Traveling Wave IM-MS (TWIMS)
  role: Source of the in-house experimental CCS values that literature database values
    are collated against for percent-deviation computation
- name: Snakemake
  role: Workflow manager whose ccs.txt output (Boltzmann-average CCS values) supplies
    the computed side of the collation table
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:c1e5a2aa888193ac40c5b944d488a84738b12e07632c123a532bbf20095a35b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CCS calculations
  - Snakemake
  - Traveling Wave IM-MS (TWIMS)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ccs-literature-value-collation

## Summary

Collates literature-reported experimental CCS values for the same metabolites and matching adduct states ([M + H]+ or [M − H]−) into a single comparison table, computes percent relative deviations, and validates against the ~±3% inter-laboratory experimental variation threshold used for predicted-vs-experimental CCS comparison.

## When to use

Use when benchmarking in-silico CCS predictions or a new instrument's TWIMS/DTIM measurements against literature CCS database values and you need a percent-deviation table keyed by metabolite and adduct state.

## When NOT to use

- The metabolites/adduct states of interest have no literature CCS database values available, so no percent-deviation collation is possible
- The input is raw instrument data (e.g., drift time or arrival-time distributions) rather than already-derived CCS values
- The analysis targets IM-MS data-collection or standardization procedures rather than value-level comparison against literature CCS

## Inputs

- TWIMS experimental CCS values and instrumental parameters (Supporting Information Tables S1 and S2)
- Boltzmann-average computed CCS values from the Snakemake workflow output ccs.txt
- Literature CCS database values for the same metabolites in protonated/deprotonated/neutral molecular states
- Benchmark metabolite list (20 metabolites, atom counts 10–40, e.g., carnosine with five molecular states)

## Outputs

- ccs_comparison.csv listing metabolite, adduct state, experimental CCS, literature CCS, consensus average, and percent deviation
- Agreement table of percent errors of calculated CCS against (a) TWIMS values, (b) literature values, and (c) the consensus average

## How to apply

Load the instrument-measured CCS values (e.g., TWIMS values with instrumental parameters from Supporting Information Tables S1 and S2) alongside the computed workflow CCS values (e.g., Boltzmann averages in ccs.txt from the Snakemake CCS workflow). Collate literature CCS database values for the same metabolites in their protonated/deprotonated/neutral molecular states into the same table. For each metabolite/adduct pair, compute the percent relative deviation between the measured TWIMS value and the literature value (and the consensus average where both exist). Tabulate metabolite, adduct state, experimental CCS, literature CCS, consensus value, and percent deviation in ccs_comparison.csv. Validate by checking that nearly all compiled deviations fall within the ~±3% threshold that the literature (e.g., 1.6% between-lab DTIM variation for the polyalanine hexamer, 190.8 vs 194.0 Å2) justifies for comparing predicted and experimental CCS.

## Related tools

- **Traveling Wave IM-MS (TWIMS)** (Source of the in-house experimental CCS values that literature database values are collated against for percent-deviation computation)
- **Snakemake** (Workflow manager whose ccs.txt output (Boltzmann-average CCS values) supplies the computed side of the collation table)

## Examples

```
python scripts/compare_ccs.py --twims Tables_S1_S2.csv --computed ensemble/ccs.txt --literature ccs_db.csv --out ccs_comparison.csv
```

## Evaluation signals

- Nearly all compiled percent deviations between measured TWIMS values and literature values fall within ±3%
- Percent errors of the calculated CCS values against literature values, TWIMS values, and consensus averages are each within ±3%
- Every row in ccs_comparison.csv has a matching adduct state between the in-house measurement and the collated literature value ([M + H]+ or [M − H]−)
- The consensus average is formed only for metabolites that have both a TWIMS value and a literature value
- Inter-laboratory deviations reproduce the reported ~±3% experimental variation threshold (cf. 1.6% between-lab DTIM variation for the polyalanine hexamer)

## Limitations

- There are no standardized procedures for IM data collection, collation, standardization, and sharing, so literature CCS values may come from heterogeneous instruments (TWIMS vs DTIM) and conditions
- Detailed comparison procedures are deferred to the Supporting Information, limiting reproducibility from the main text alone
- Most metabolites lack authentic chemical standards, so literature CCS values may not be available or authoritative for many compounds

## Evidence

- [other] Collate the literature-reported CCS values for the same metabolites and matching adduct states ([M + H]+ or [M − H]−) into the same table.: [evidence span withheld — anchor status: not_found]
- [other] verify that the inter-laboratory deviations reproduce the reported approximately ±3% experimental variation threshold: [evidence span withheld — anchor status: not_found]
- [other] Published DTIM CCS measurements of the polyalanine hexamer differ between laboratories by 1.6% (190.8 vs 194.0 Å2): [evidence span withheld — anchor status: not_found]
- [intro] The CCS values calculated using our workflow were in good agreement with available experimental values in the literature (±3%): "The CCS values calculated using our workflow were in good agreement with available experimental values in the literature (±3%), with the values determined herein (±3%)"
- [intro] there are no standardized procedures for data collection, collation, standardization, and sharing.: "there are no standardized procedures for data collection, collation, standardization, and sharing."
