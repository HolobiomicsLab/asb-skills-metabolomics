---
name: msms-query-spectrum-processing
description: Use when MS/MS query spectra from an annotated molecular network (GNPS/MassIVE)
  need per-spectrum molecular formula and structure identification, or when benchmarking
  a new annotation method against SIRIUS.
when_to_use_negative:
- Input is raw LC-MS/MS acquisition data rather than deconvoluted, library-annotated
  query spectra from an annotated molecular network node
- The query compound is not represented in the structural database (bioDB), since
  annotation is only evaluated for compounds present there
- The goal is de novo biotransformation-based annotation of unknown suspects (use
  BAM instead) rather than database-restricted formula/structure identification
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: 'Command-line tool used for two-stage annotation: molecular formula prediction
    against bioDB, followed by molecular structure prediction against bioDB only for
    spectra with a correct formula'
- name: BAM
  role: Provides the anchor–suspect query set and input CSVs (molecules_of_interest)
    that the spectra processing pipeline consumes; run via runBAM.sh with PROXIMAL2
    and GNN-SOM conda environments
  repo: https://github.com/HassounLab/BAM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:3c15345ea77b270ae069989e213eac5ecc929a34f9cd7d36115050b906944d46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# msms-query-spectrum-processing

## Summary

Prepare MS/MS query spectra from a global molecular network anchor–suspect data set and run SIRIUS 5.6.3 molecular formula and structure annotation against bioDB, establishing a baseline annotation rate (4756 correct formulas / 4560 correct structures out of 38,368 queries).

## When to use

Use when MS/MS query spectra from an annotated molecular network (GNPS/MassIVE) need per-spectrum molecular formula and structure identification, or when benchmarking a new annotation method against SIRIUS.

## When NOT to use

- Input is raw LC-MS/MS acquisition data rather than deconvoluted, library-annotated query spectra from an annotated molecular network node
- The query compound is not represented in the structural database (bioDB), since annotation is only evaluated for compounds present there
- The goal is de novo biotransformation-based annotation of unknown suspects (use BAM instead) rather than database-restricted formula/structure identification

## Inputs

- 38,368 unique MS/MS query spectra from the anchor–suspect data set derived from the global molecular network (filtered for compounds present in bioDB)
- GNPS/MassIVE metadata per spectrum: adduct, ionization mode, instrument type
- bioDB biomolecule structure database

## Outputs

- Prediction table with one row per query spectrum (38,368 rows)
- Per-query molecular formula annotations from SIRIUS (4756 correct)
- Per-query molecular structure annotations from SIRIUS for formula-correct spectra (4560 correct)

## How to apply

Load the unique query spectra filtered for compounds present in the structural database (bioDB in this paper, derived from the 60,368 anchor–suspect pairs of the global molecular network). Configure SIRIUS 5.6.3 through its CLI, pulling adduct, ionization mode, and instrument type from the corresponding GNPS/MassIVE metadata, with default elements (C, H, N, O, P, S, Br, Cl, B, Se) and 10 ppm mass accuracy. Run molecular formula annotation against bioDB first, and only if the correct formula is generated, run molecular structure annotation — this two-stage gate is the paper's stated decision point. Aggregate per-query results into a prediction table covering every query spectrum, then count true formulas and true structures. Validate that the table has one row per query and that counts reproduce the reference values.

## Related tools

- **SIRIUS** (Command-line tool used for two-stage annotation: molecular formula prediction against bioDB, followed by molecular structure prediction against bioDB only for spectra with a correct formula)
- **BAM** (Provides the anchor–suspect query set and input CSVs (molecules_of_interest) that the spectra processing pipeline consumes; run via runBAM.sh with PROXIMAL2 and GNN-SOM conda environments) — https://github.com/HassounLab/BAM

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- Prediction table row count must equal 38,368 (one row per query spectrum)
- True molecular formula count must reproduce the reference value 4756
- True molecular structure count must reproduce the reference value 4560
- Structure annotation is run only on spectra where the correct molecular formula was generated (two-stage gating invariant)
- Adduct, ionization mode, and instrument type settings must match the GNPS/MassIVE metadata for each spectrum

## Limitations

- Annotation is restricted to the structural database (bioDB); queries whose true compound is absent from the database cannot be correctly identified
- Only a small minority of MS/MS spectra in untargeted metabolomics receive annotations — existing methodologies fall short in interpreting the vast majority of MS/MS data
- Structure annotation is conditional on formula correctness, so formula errors propagate and suppress structure-level results

## Evidence

- [methods] SIRIUS applied to anchor–suspect query spectra: [evidence span withheld — anchor status: not_found]
- [methods] Two-stage formula-then-structure annotation gate: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [methods] SIRIUS CLI configuration parameters: [evidence span withheld — anchor status: not_found]
- [methods] Query spectra must be filtered to bioDB compounds: [evidence span withheld — anchor status: not_found]
- [readme] Query input file format for the pipeline: "molecules_of_interest = csv file of list of queries. It must have an identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor."
