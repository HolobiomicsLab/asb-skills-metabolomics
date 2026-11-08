---
name: inter-laboratory-ccs-variation-analysis
description: Use when comparing TWIMS (or DTIM) CCS values measured in-house against
  literature CCS values for the same metabolite/adduct ([M + H]+ or [M − H]−), or
  when validating that predicted CCS values agree with experimental ones within the
  ~±3% experimental variation band.
when_to_use_negative:
- The goal is computing CCS de novo from SMILES structures (use the Snakemake CCS
  prediction workflow with RDKit/ASE-ANI/QUICK/hpccs instead of a literature-variation
  comparison)
- The metabolites lack any literature or consensus CCS values for the matching adduct
  state, leaving no inter-laboratory pair to compare
- Comparing CCS values from the same instrument and laboratory run, where run-to-run
  repeatability rather than between-laboratory variation is the quantity of interest
tools:
- name: Traveling Wave IM-MS (TWIMS)
  role: Instrument used to measure the in-house experimental CCS values that are compared
    against literature and consensus values to assess uncertainty
- name: Snakemake CCS workflow (snakemake_ccs)
  role: Produces the predicted (Boltzmann average) CCS values whose agreement with
    experimental values is judged against the ~±3% threshold
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:9d6f2ff2ab8b893934a2a783ad03dfcf7e8c24aa91bea5822bb0967ee8b6c705
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/DasSusanta/snakemake_ccs
  tools:
  - Snakemake CCS workflow (snakemake_ccs)
  - Traveling Wave IM-MS (TWIMS)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# inter-laboratory-ccs-variation-analysis

## Summary

Quantifies how much experimentally measured collision cross section (CCS) values vary between laboratories and instruments (e.g., TWIMS vs DTIM) by computing percent relative deviations between locally measured and literature-reported CCS values, and uses the resulting spread to justify an approximately ±3% acceptance threshold for predicted-vs-experimental CCS comparison.

## When to use

Use when comparing TWIMS (or DTIM) CCS values measured in-house against literature CCS values for the same metabolite/adduct ([M + H]+ or [M − H]−), or when validating that predicted CCS values agree with experimental ones within the ~±3% experimental variation band.

## When NOT to use

- The goal is computing CCS de novo from SMILES structures (use the Snakemake CCS prediction workflow with RDKit/ASE-ANI/QUICK/hpccs instead of a literature-variation comparison)
- The metabolites lack any literature or consensus CCS values for the matching adduct state, leaving no inter-laboratory pair to compare
- Comparing CCS values from the same instrument and laboratory run, where run-to-run repeatability rather than between-laboratory variation is the quantity of interest

## Inputs

- TWIMS CCS experimental values with instrumental parameters (Supporting Information Tables S1 and S2)
- Literature-reported CCS values for matching metabolite/adduct pairs ([M + H]+ or [M − H]−)
- Consensus average CCS values where available
- Predicted CCS values to benchmark (e.g., Boltzmann average values from the Snakemake workflow ccs.txt output)

## Outputs

- ccs_comparison.csv with columns: metabolite, adduct state, experimental CCS, literature CCS, percent deviation
- Estimate of inter-laboratory experimental CCS variation (percent), supporting a ~±3% acceptance threshold

## How to apply

Load the TWIMS CCS experimental values and instrumental parameters from Supporting Information Tables S1 and S2. Collate literature-reported CCS values for the same metabolites and matching adduct states ([M + H]+ or [M − H]−) into one table. For each metabolite/adduct pair, compute the percent relative deviation between the TWIMS value measured in this work and the literature value (and the consensus average where available). Tabulate metabolite, adduct state, experimental CCS, literature CCS, and percent deviation in ccs_comparison.csv. Validation: confirm nearly all compiled deviations fall within ±3%, reproducing the reported experimental variation threshold; the article reports workflow CCS values in good agreement with literature values (±3%), values determined herein (±3%), and consensus averages.

## Related tools

- **Traveling Wave IM-MS (TWIMS)** (Instrument used to measure the in-house experimental CCS values that are compared against literature and consensus values to assess uncertainty)
- **Snakemake CCS workflow (snakemake_ccs)** (Produces the predicted (Boltzmann average) CCS values whose agreement with experimental values is judged against the ~±3% threshold) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
import pandas as pd; df = pd.read_csv('ccs_pairs.csv'); df['pct_dev'] = 100*(df['twims_ccs']-df['lit_ccs'])/df['lit_ccs']; assert (df['pct_dev'].abs()<=3).all(); df.to_csv('ccs_comparison.csv', index=False)
```

## Evaluation signals

- ccs_comparison.csv contains one row per metabolite/adduct pair with experimental CCS, literature CCS, and percent deviation columns populated
- Nearly all computed percent relative deviations fall within ±3%, reproducing the reported approximately ±3% experimental variation threshold
- Adduct states match exactly between the in-house and literature values ([M + H]+ compared only against [M + H]+; [M − H]− only against [M − H]−)
- For metabolites with multiple literature measurements (e.g., between-lab spread such as 190.8 vs 194.0 Å2, ~1.6%), the compiled deviations bracket that reported between-laboratory variation
- Predicted/consensus comparisons agree with literature values (±3%), values determined herein (±3%), and consensus averages as reported

## Limitations

- There are no standardized procedures for IM data collection, collation, standardization, and sharing, so literature CCS values may not be directly comparable across instruments
- The ±3% band is an empirical acceptance threshold justified by observed between-laboratory variation, not a formal statistical tolerance interval
- In this study the comparison is limited to 20 metabolites with atom counts from 10 to 40; extrapolating the ±3% threshold to much larger or more conformationally flexible molecules is not validated here

## Evidence
