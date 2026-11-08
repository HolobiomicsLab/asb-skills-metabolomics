---
name: massive-public-dataset-spectrum-search-retrieval
description: Use when newly synthesized reference compounds (e.g. N-acyl amides, acyl
  esters) have MS/MS spectra that must be searched repository-wide in public metabolomics
  data and per-compound match counts compiled.
when_to_use_negative:
- You have no tandem MS/MS spectra for the target compounds — reverse metabolomics
  via MASST requires an MS/MS spectrum, and metabolites must ionize well enough to
  be selected for MS/MS acquisition
- You need de novo structural elucidation — MASST matching does not exclude different
  isomers without isolation and NMR or X-ray structural analyses
- The target molecules are absent from public spectral repositories — MASST's ~1.2
  billion spectra do not encompass all molecules, limiting re-analysis of public data
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MASST
  role: Searches compound MS/MS spectra (via USI) against public metabolomics datasets
    repository-scale and returns per-compound spectral match records
- name: Nextflow
  role: Runs the library-generation workflow used to produce/deploy the GNPS2 workflows
    underlying the synthetic library spectra
  repo: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
- name: conda/mamba
  role: Environment manager for installing dependencies to run the library-generation
    and deployment workflows locally
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:e539c994a961ab0ba7488c39254b7b45a955fce176c5e7b4824c144b650f6656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
  tools:
  - MASST
  - Nextflow
  - conda/mamba
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Repository-scale MASST spectral search retrieval

## Summary

Search synthesized compound MS/MS spectra (delivered via USI) against public metabolomics repositories with MASST, then parse and aggregate per-compound match records to quantify how many public files and spectra contain each target molecule class.

## When to use

Use when newly synthesized reference compounds (e.g. N-acyl amides, acyl esters) have MS/MS spectra that must be searched repository-wide in public metabolomics data and per-compound match counts compiled.

## When NOT to use

- You have no tandem MS/MS spectra for the target compounds — reverse metabolomics via MASST requires an MS/MS spectrum, and metabolites must ionize well enough to be selected for MS/MS acquisition
- You need de novo structural elucidation — MASST matching does not exclude different isomers without isolation and NMR or X-ray structural analyses
- The target molecules are absent from public spectral repositories — MASST's ~1.2 billion spectra do not encompass all molecules, limiting re-analysis of public data

## Inputs

- Synthetic-mixture MS/MS spectra referenced by USI (e.g. from MSV000089491 and MSV000089493)
- MASST GNPS task result links / job identifiers for each synthesized compound
- Per-compound mapping of USI to MASST job identifier

## Outputs

- per_compound_match_tables.csv with per-compound match records
- Aggregate counts: total spectral matches, matched unique files, and fraction of compounds returning matches, per compound class

## How to apply

Retrieve the deposited synthetic-mixture spectra for each compound class (e.g. dataset MSV000089491 for N-acyl amides, MSV000089493 for acyl esters) via their USI and the published MASST GNPS task job identifiers. Parse each MASST job's match records, keyed per synthesized compound through its USI and MASST job identifier. Aggregate matches per compound: count matched public files and total spectral matches per compound class. Write per-compound match tables to per_compound_match_tables.csv. Reconcile compiled totals against reported figures (amides: 60,277 matches across 25,463 files with 31% of compounds matched; esters: 5,884 matches across 5,273 files with 13 esters detected) and accept the tables only when totals match.

## Related tools

- **MASST** (Searches compound MS/MS spectra (via USI) against public metabolomics datasets repository-scale and returns per-compound spectral match records)
- **Nextflow** (Runs the library-generation workflow used to produce/deploy the GNPS2 workflows underlying the synthetic library spectra) — https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
- **conda/mamba** (Environment manager for installing dependencies to run the library-generation and deployment workflows locally)

## Evaluation signals

- Amide-target totals in per_compound_match_tables.csv reconcile to 60,277 spectral matches across 25,463 unique files with 31% of searched compounds matched
- Ester-target totals reconcile to 5,884 spectral matches across 5,273 files with 13 esters detected
- Every match record can be keyed back to a synthesized compound through its USI and MASST job identifier
- Match counts per compound and per class are internally consistent (sum of per-compound matches equals the class-level total)

## Limitations

- MASST searches encompass ~1.2 billion spectra but do not include all molecules, limiting discovery-phase re-analysis of public data
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, though it prevents completely inaccurate identifications
- Without isolation and NMR or X-ray structural analyses, different isomers represented by the matched data cannot be excluded
- Metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition

## Evidence

- [other] workflow step quoting MASST results: [evidence span withheld — anchor status: not_found]
- [other] workflow aggregation step: [evidence span withheld — anchor status: not_found]
- [other] validation criterion: [evidence span withheld — anchor status: not_found]
- [other] key finding quantifying matches: [evidence span withheld — anchor status: not_found]
- [abstract] workflow definition: "tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [discussion] repository coverage limitation: "the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules. This limits re-analyses of public data in the discovery phase of this approach."
- [readme] local execution prerequisites: "You will need to have conda, mamba, and nextflow installed to run things locally."
