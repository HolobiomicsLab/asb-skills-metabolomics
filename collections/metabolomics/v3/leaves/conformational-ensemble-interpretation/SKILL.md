---
name: conformational-ensemble-interpretation
description: Use when a ccs.txt output from the Snakemake CCS workflow (ensemble or
  ensemble_fast folder) exists and the goal is to judge whether ensemble averaging
  outperforms the global-minimum conformer against experimental TWIMS CCS values.
when_to_use_negative:
- The task is to run the CCS workflow itself (SMILES to ccs.txt); this skill starts
  only after the Snakemake workflow has completed.
- Only a single conformational state or a rigid molecule is under study, so ensemble
  averaging versus global-minimum comparison is not meaningful.
- No experimental TWIMS CCS reference values are available for the metabolites, so
  percent-error validation cannot be performed.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3332
tools:
- name: Snakemake
  role: Workflow manager whose output ccs.txt (Boltzmann average CCS) is the primary
    input for this skill
  repo: https://github.com/DasSusanta/snakemake_ccs
- name: HPCCS
  role: Produces per-conformer CCS values and trajectory outputs used to identify
    the global-minimum conformer
  repo: https://github.com/cepid-cces/hpccs
- name: RDKit
  role: Generates the initial conformations that form the ensemble being interpreted
  repo: https://www.rdkit.org
- name: ASE-ANI
  role: Filters conformations to the ensemble of structures evaluated by HPCCS
  repo: https://github.com/isayev/ASE_ANI
- name: QUICK
  role: Performs the quantum calculations that supply conformer relative energies
    used for Boltzmann weighting
  repo: https://github.com/merzlab/QUICK
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:78502101d0c9e665f72d29e04cab8a9794aa3a51749bb31f048f65cd6042950a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/DasSusanta/snakemake_ccs
  tools:
  - ASE-ANI
  - HPCCS
  - QUICK
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# conformational-ensemble-interpretation

## Summary

Interprets the output of a quantum-chemistry CCS workflow by comparing Boltzmann-averaged CCS values from the full conformational ensemble against the CCS of the single global-minimum conformer, to decide which predictor best matches experimental TWIMS values.

## When to use

Use when a ccs.txt output from the Snakemake CCS workflow (ensemble or ensemble_fast folder) exists and the goal is to judge whether ensemble averaging outperforms the global-minimum conformer against experimental TWIMS CCS values.

## When NOT to use

- The task is to run the CCS workflow itself (SMILES to ccs.txt); this skill starts only after the Snakemake workflow has completed.
- Only a single conformational state or a rigid molecule is under study, so ensemble averaging versus global-minimum comparison is not meaningful.
- No experimental TWIMS CCS reference values are available for the metabolites, so percent-error validation cannot be performed.

## Inputs

- ccs.txt file with Boltzmann average CCS values per metabolite (from the ensemble or ensemble_fast folder of the snakemake_ccs workflow)
- HPCCS trajectory outputs containing individual conformer CCS values and relative energies
- Experimental TWIMS CCS reference values from Supporting Information Tables S1 and S2

## Outputs

- ccs_averaging_comparison.csv with per-metabolite percent errors for Boltzmann-averaged vs global-minimum-conformer CCS
- Summary statistics comparing the two error distributions against the ±3% agreement criterion

## How to apply

Parse the ccs.txt file in the ensemble (or ensemble_fast) folder to obtain the Boltzmann average CCS per metabolite, then extract per-conformer CCS values and relative energies from the HPCCS trajectory outputs to identify the global-minimum conformer for each metabolite. Load experimental TWIMS CCS reference values from Supporting Information Tables S1 and S2. For each of the 20 benchmark metabolites compute the percent error of both the Boltzmann-averaged CCS and the global-minimum-conformer CCS against the experimental TWIMS value, then compare the two error distributions using the paper's ±3% agreement criterion. Write the per-metabolite error table and summary statistics to ccs_averaging_comparison.csv. The rationale is that conformationally flexible metabolites sample multiple structures, so ensemble weighting (Boltzmann averaging over conformers) can predict experimental CCS better than relying on the energetically lowest conformer alone — e.g. for model 3 of L-tryptophan the Boltzmann average of 159.69 Å² predicted the experimental CCS of 154.22 Å² better than the global-minimum conformer's 166.44 Å².

## Related tools

- **Snakemake** (Workflow manager whose output ccs.txt (Boltzmann average CCS) is the primary input for this skill) — https://github.com/DasSusanta/snakemake_ccs
- **HPCCS** (Produces per-conformer CCS values and trajectory outputs used to identify the global-minimum conformer) — https://github.com/cepid-cces/hpccs
- **RDKit** (Generates the initial conformations that form the ensemble being interpreted) — https://www.rdkit.org
- **ASE-ANI** (Filters conformations to the ensemble of structures evaluated by HPCCS) — https://github.com/isayev/ASE_ANI
- **QUICK** (Performs the quantum calculations that supply conformer relative energies used for Boltzmann weighting) — https://github.com/merzlab/QUICK

## Examples

```
bash scheduler.sh  # then parse ensemble/ccs.txt and HPCCS outputs to compute per-metabolite percent errors vs TWIMS values in Tables S1/S2
```

## Evaluation signals

- For each of the 20 benchmark metabolites, percent errors for both CCS predictors are computed against experimental TWIMS values (Tables S1/S2)
- Boltzmann-averaged CCS matches experimental values within the paper's ±3% criterion
- ccs_averaging_comparison.csv contains one row per metabolite with both Boltzmann-average and global-minimum-conformer percent errors
- Known case reproduced: for model 3 of L-tryptophan, Boltzmann average CCS of 159.69 Å² vs experimental 154.22 Å² outperforms the global-minimum conformer CCS of 166.44 Å²
- Boltzmann averaging yields a systematically lower error distribution than the single global-minimum conformer across the benchmark set

## Limitations

- Boltzmann averaging does not always win — for L-tryptophan (model 3) the averaged CCS (159.69 Å²) was still ~3.5% above the experimental 154.22 Å², though better than the global-minimum conformer (166.44 Å²)
- Accuracy depends on the ensemble quality; ANI-1x/ANI-ccx potentials used for conformation filtering cover only CHNO elements, and the older ASE-ANI interface is deprecated in favor of TorchANI
- Results are calibrated on a 20-metabolite benchmark with atom counts of 10–40; conformationally complex or larger molecules may behave differently

## Evidence
