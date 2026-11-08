---
name: twims-ccs-measurement-extraction
description: Use when TWIMS CCS experimental values and instrumental parameters are
  available in Supporting Information tables and the goal is to compare them against
  literature CCS values for the same metabolites and adduct states to establish an
  experimental uncertainty threshold.
when_to_use_negative:
- Input is predicted/in silico CCS output from the Snakemake CCS workflow rather than
  experimentally measured TWIMS values — use a prediction-validation comparison instead
  of inter-laboratory deviation collation
- The adduct states of the literature and in-house CCS values do not match ([M + H]+
  vs [M − H]−), since percent deviation is only meaningful for the same ion species
- The goal is measuring new CCS values on an instrument rather than collating and
  comparing already-measured TWIMS values against literature values
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Traveling Wave IM-MS (TWIMS)
  role: Instrument used to experimentally measure CCS values for the 20 benchmark
    metabolites, whose measurements are collated against literature values
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:d17d62031f4a3d4baccdaef20d2f48a727e313e8c88f3c5939ed8054e62d0829
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Traveling Wave IM-MS (TWIMS)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# twims-ccs-measurement-extraction

## Summary

Extracts experimentally measured TWIMS CCS values together with instrumental parameters from Supporting Information Tables S1/S2, then collates matching literature CCS values per metabolite/adduct state to quantify inter-laboratory deviation and validate the ~±3% experimental variation threshold.

## When to use

Use when TWIMS CCS experimental values and instrumental parameters are available in Supporting Information tables and the goal is to compare them against literature CCS values for the same metabolites and adduct states to establish an experimental uncertainty threshold.

## When NOT to use

- Input is predicted/in silico CCS output from the Snakemake CCS workflow rather than experimentally measured TWIMS values — use a prediction-validation comparison instead of inter-laboratory deviation collation
- The adduct states of the literature and in-house CCS values do not match ([M + H]+ vs [M − H]−), since percent deviation is only meaningful for the same ion species
- The goal is measuring new CCS values on an instrument rather than collating and comparing already-measured TWIMS values against literature values

## Inputs

- TWIMS CCS experimental values (Supporting Information, Tables S1 and S2)
- Instrumental parameters for the TWIMS measurements (Supporting Information Tables S1 and S2)
- Literature-reported CCS values for the same metabolites and matching adduct states ([M + H]+ or [M − H]−)
- Consensus average CCS values where available

## Outputs

- ccs_comparison.csv with columns: metabolite, adduct state, experimental CCS, literature CCS, percent deviation
- Validated estimate of inter-laboratory experimental CCS variation (approximately ±3%)

## How to apply

Load the TWIMS CCS experimental values and the accompanying instrumental parameters from Supporting Information Tables S1 and S2. Collate literature-reported CCS values for the same metabolites and matching adduct states ([M + H]+ or [M − H]−) into a single table, ensuring each comparison pair refers to the same ion species. For each metabolite/adduct pair, compute the percent relative deviation between the TWIMS value measured in this work and the literature value, and the consensus average where available. Tabulate metabolite, adduct state, experimental CCS, literature CCS, and percent deviation in ccs_comparison.csv. Validate by checking that nearly all compiled deviations fall within the approximately ±3% experimental variation threshold reported in the article, which was justified by published between-laboratory differences such as the 1.6% (190.8 vs 194.0 Å2) DTIM variation for the polyalanine hexamer.

## Related tools

- **Traveling Wave IM-MS (TWIMS)** (Instrument used to experimentally measure CCS values for the 20 benchmark metabolites, whose measurements are collated against literature values)

## Examples

```
python collate_ccs.py --twims si_tables_S1_S2.csv --literature lit_ccs.csv --adducts '[M+H]+,[M-H]-' --output ccs_comparison.csv
```

## Evaluation signals

- Nearly all compiled percent deviations between TWIMS values and literature values fall within approximately ±3%
- ccs_comparison.csv contains metabolite, adduct state, experimental CCS, literature CCS, and percent deviation for every collated pair
- Each comparison pair is restricted to matching adduct states ([M + H]+ or [M − H]−)
- Computed CCS values agree within ±3% with literature experimental values, values determined in this study, and consensus averages
- Inter-laboratory deviations are consistent with reported between-lab variation (e.g. 1.6% for DTIM CCS of the polyalanine hexamer, 190.8 vs 194.0 Å2)

## Limitations

- Detailed workflow step descriptions are deferred to the Supporting Information, so extraction relies on tabulated values rather than fully described in-text procedures
- No standardized procedures exist for IM data collection, collation, standardization, and sharing, which complicates cross-study CCS comparisons
- The article's field gap: the vast majority of metabolites do not have authentic chemical standards, limiting the availability of reference experimental CCS values

## Evidence
