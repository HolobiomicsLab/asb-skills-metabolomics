---
name: adduct-state-matching-for-ccs-comparison
description: Use when experimentally measured CCS values (e.g., TWIMS from Supporting
  Information Tables S1/S2) must be compared against literature CCS values per metabolite/adduct
  pair to quantify between-laboratory deviation or justify a ±3% acceptance threshold.
when_to_use_negative:
- Comparing predicted (in silico) CCS values to experimental values when the adduct/protonation
  state of the ion is unknown or ambiguous — the comparison is only valid when the
  same ion state is matched on both sides
- Datasets where literature CCS values lack adduct-state metadata, making metabolite/adduct
  pairing impossible without re-derivation
- Estimating absolute instrument calibration error rather than between-laboratory/literature
  variation — this skill quantifies percent relative deviation against published values,
  not calibration curves
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Traveling Wave IM-MS (TWIMS)
  role: Instrument used to measure the experimental CCS values that are compared against
    literature values per metabolite/adduct pair
- name: Dimorphite-DL
  role: Determines ionization/protonation states of metabolites so that CCS comparisons
    use the correct adduct states
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: Snakemake CCS workflow (DasSusanta/snakemake_ccs)
  role: Generates the computed CCS values per protonated/deprotonated adduct that
    are matched against experimental and literature values
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:f167b3971f0c0d328b9789acfc41585d23f9fba61082af4aa278f670b85b6eb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Dimorphite-DL
  - Snakemake CCS workflow (DasSusanta/snakemake_ccs)
  - Traveling Wave IM-MS (TWIMS)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# adduct-state-matching-for-ccs-comparison

## Summary

Collates experimentally measured TWIMS CCS values with literature-reported CCS values for the same metabolites under matching adduct states ([M + H]+ or [M − H]−), computes percent relative deviations, and validates them against the ~±3% inter-laboratory experimental variation threshold.

## When to use

Use when experimentally measured CCS values (e.g., TWIMS from Supporting Information Tables S1/S2) must be compared against literature CCS values per metabolite/adduct pair to quantify between-laboratory deviation or justify a ±3% acceptance threshold.

## When NOT to use

- Comparing predicted (in silico) CCS values to experimental values when the adduct/protonation state of the ion is unknown or ambiguous — the comparison is only valid when the same ion state is matched on both sides
- Datasets where literature CCS values lack adduct-state metadata, making metabolite/adduct pairing impossible without re-derivation
- Estimating absolute instrument calibration error rather than between-laboratory/literature variation — this skill quantifies percent relative deviation against published values, not calibration curves

## Inputs

- TWIMS CCS experimental values with instrumental parameters (Supporting Information Tables S1 and S2)
- Literature-reported CCS values for the same metabolites with adduct-state annotations ([M + H]+ or [M − H]−)
- Metabolite identities and their protonated/deprotonated/neutral molecular states

## Outputs

- ccs_comparison.csv with columns: metabolite, adduct state, experimental CCS, literature CCS, percent deviation
- Assessment that inter-laboratory/inter-source deviations fall within the ~±3% threshold

## How to apply

Load the TWIMS CCS experimental values and instrumental parameters from Supporting Information Tables S1 and S2. Collate literature-reported CCS values for the same metabolites into the same table, matching adduct/protonation states explicitly (typically [M + H]+ or [M − H]−) so that computed and experimental values refer to the same gas-phase ion. For each metabolite/adduct pair, compute the percent relative deviation between the TWIMS value measured in the work and the literature value, and where available the consensus average. Tabulate metabolite, adduct state, experimental CCS, literature CCS, and percent deviation in ccs_comparison.csv. Validate by checking that nearly all compiled deviations fall within the approximately ±3% experimental variation threshold reported in the article.

## Related tools

- **Traveling Wave IM-MS (TWIMS)** (Instrument used to measure the experimental CCS values that are compared against literature values per metabolite/adduct pair)
- **Dimorphite-DL** (Determines ionization/protonation states of metabolites so that CCS comparisons use the correct adduct states) — https://durrantlab.pitt.edu/dimorphite-dl
- **Snakemake CCS workflow (DasSusanta/snakemake_ccs)** (Generates the computed CCS values per protonated/deprotonated adduct that are matched against experimental and literature values) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
python compare_ccs.py --experimental SI_TableS1_twims.csv --literature lit_ccs.csv --adduct-col adduct --output ccs_comparison.csv --tolerance 3.0
```

## Evaluation signals

- Nearly all computed percent relative deviations between TWIMS and literature CCS values fall within ±3%
- Every row of ccs_comparison.csv pairs experimental and literature CCS values for the same metabolite AND the same adduct state (e.g., [M + H]+ vs [M + H]+)
- Inter-laboratory variation reproduced (e.g., DTIM polyalanine hexamer values differing by 1.6% between laboratories: 190.8 vs 194.0 Å2) supports the ±3% threshold as justified
- Consensus averages, where available, agree with both the in-house TWIMS values and literature values within the same ±3% window

## Limitations

- There are no standardized procedures for IM data collection, collation, standardization, and sharing, so literature CCS values may come from different instruments (TWIMS vs DTIM) and conditions
- The workflow's detailed step descriptions are deferred to the Supporting Information, limiting reproducibility of the comparison protocol from the main text alone
- Adduct-state determination relies on assumed typical gas-phase ions ([M + H]+ or [M − H]−); metabolites forming multiple protonation states (e.g., carnosine with three monoprotonated and one doubly protonated ion) require explicit state-level matching

## Evidence

- [methods] Collate literature CCS values per metabolite with matching adduct states: [evidence span withheld — anchor status: not_found]
- [methods] Determine protonation state before comparison: "Initially, we determine the molecular protonation state of the corresponding gas-phase ions (typically [M + H]+ or [M − H]−)"
- [intro] ±3% agreement threshold with literature values: "The CCS values calculated using our workflow were in good agreement with available experimental values in the literature (±3%), with the values determined herein (±3%)"
- [methods] TWIMS experimental values and parameters in SI Tables S1/S2: "The TWIMS CCS experimental values are also reported in the Supporting Information, together with instrumental parameters (Tables S1 and S2)."
- [results] Carnosine multiple protonation states: "for carnosine we have generated five states including one neutral, three monoprotonated ions, and one doubly protonated ion"
