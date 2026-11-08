---
name: ion-mobility-ccs-experimental-comparison
description: Use when per-conformer or Boltzmann-averaged CCS values (e.g., from HPCCS
  trajectory calculations) need to be validated against TWIMS experimental CCS references,
  or when ranking conformers to find the one matching the measured CCS.
when_to_use_negative:
- The molecule has no available experimental CCS reference (no TWIMS measurement in
  Tables S1/S2, literature, or databases like MassBank), so no comparison target exists
- The goal is CCS prediction itself from a SMILES input rather than validation/ranking
  against measured values — use the upstream prediction workflow instead
- The experimental value was measured on an instrument type other than TWIMS without
  the ±3% uncertainty assessment supporting cross-method comparison
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: HPCCS
  role: Produces the per-conformer CCS values (room-temperature N2-based trajectory
    method) that are compared against experimental TWIMS CCS values
  repo: https://github.com/cepid-cces/hpccs
- name: TWIMS
  role: Traveling Wave IM-MS instrument providing the experimental CCS measurements
    used as the comparison reference (Tables S1/S2)
- name: Snakemake workflow (snakemake_ccs)
  role: Orchestrates the pipeline whose final output, ccs.txt in the ensemble folder,
    contains the Boltzmann-average CCS values used for the experimental comparison
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:d398f42650e9cdf250a7d8d0f95d9821294d5901d785c91397928bc3ecaf2da9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cepid-cces/hpccs
  tools:
  - HPCCS
  - Snakemake workflow (snakemake_ccs)
  - TWIMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ion-mobility-ccs-experimental-comparison

## Summary

Compares computed collision cross section (CCS) values (per-conformer and Boltzmann-averaged) against experimental TWIMS reference values from literature or Supporting Information tables (e.g., Tables S1/S2) to validate a CCS prediction workflow and identify the conformer whose CCS best matches experiment.

## When to use

Use when per-conformer or Boltzmann-averaged CCS values (e.g., from HPCCS trajectory calculations) need to be validated against TWIMS experimental CCS references, or when ranking conformers to find the one matching the measured CCS.

## When NOT to use

- The molecule has no available experimental CCS reference (no TWIMS measurement in Tables S1/S2, literature, or databases like MassBank), so no comparison target exists
- The goal is CCS prediction itself from a SMILES input rather than validation/ranking against measured values — use the upstream prediction workflow instead
- The experimental value was measured on an instrument type other than TWIMS without the ±3% uncertainty assessment supporting cross-method comparison

## Inputs

- Per-conformer CCS values from HPCCS trajectory calculations (e.g., 166.44 and 156.03 Å² for L-tryptophan conformations 4 and 9)
- Boltzmann weights over the DFT (B3LYP/6-311++G(d,p)) conformer ensemble
- Experimental TWIMS CCS reference values (Supporting Information Tables S1/S2) or literature/database CCS values
- Computed per-conformer and Boltzmann-average CCS values written to ccs.txt in the ensemble folder

## Outputs

- Agreement assessment of computed vs. experimental CCS within the stated ±3% tolerance
- Ranking of conformers identifying the preferred conformer whose CCS best matches the TWIMS measurement
- ccs.txt file with Boltzmann-average CCS and per-conformer results

## How to apply

After running the room-temperature N2-based trajectory method in HPCCS for each conformer, compute the Boltzmann-weighted average CCS over the ensemble (e.g., for L-tryptophan [M+H]+ model 3 this yields 159.69 Å² from conformations 4 and 9, at 166.44 and 156.03 Å²). Collect the experimental TWIMS CCS reference from the Supporting Information tables (Tables S1/S2) or from literature/consensus values. Compare computed values against experiment under the workflow's stated ±3% agreement criterion, and rank the per-conformer CCS values to identify which conformer's CCS best matches the TWIMS measurement (conformer 3 for L-tryptophan). Judge correctness both by whether the Boltzmann average reproduces the reference within ±3% and by whether the ranking reveals a preferred conformer, reporting results (e.g., Boltzmann-average CCS and per-conformer values) to ccs.txt in the ensemble folder.

## Related tools

- **HPCCS** (Produces the per-conformer CCS values (room-temperature N2-based trajectory method) that are compared against experimental TWIMS CCS values) — https://github.com/cepid-cces/hpccs
- **TWIMS** (Traveling Wave IM-MS instrument providing the experimental CCS measurements used as the comparison reference (Tables S1/S2))
- **Snakemake workflow (snakemake_ccs)** (Orchestrates the pipeline whose final output, ccs.txt in the ensemble folder, contains the Boltzmann-average CCS values used for the experimental comparison) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
bash scheduler.sh  # run the Snakemake CCS workflow; then compare ccs.txt Boltzmann-average CCS (e.g., 159.69 Å² for L-tryptophan) to the TWIMS value (154.22 Å²) within ±3%
```

## Evaluation signals

- Computed CCS values agree with experimental values within the workflow's stated ±3% agreement criterion
- For L-tryptophan [M+H]+ model 3, the Boltzmann-average CCS reproduces the reported 159.69 Å² (from conformations 4 and 9: 166.44 and 156.03 Å²)
- The experimental reference for protonated L-tryptophan is 154.22 Å², and the Boltzmann average outperforms the global-minimum conformer alone
- Conformer ranking identifies conformer 3 as the conformer whose CCS best matches the TWIMS measurement
- ccs.txt is generated in the ensemble (or ensemble_fast) folder containing the Boltzmann average CCS values

## Limitations

- Comparison accuracy depends on validated experimental values; the authors performed TWIMS experiments specifically to validate extant experimental values and assess uncertainties, since no standardized procedures exist for IM data collection, collation, standardization, and sharing
- For flexible metabolites, the single global-minimum conformer can give worse agreement than the Boltzmann average, so per-conformer comparison alone can mislead (e.g., L-tryptophan model 3)

## Evidence
