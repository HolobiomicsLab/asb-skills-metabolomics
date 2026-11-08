---
name: masst-job-result-parsing
description: Use when you hold MASST GNPS task result links (with USIs and MASST job
  identifiers) for MS/MS spectra of synthesized compounds and need per-compound counts
  of spectral matches and matched public files, e.g. to quantify how widely a synthetic
  library re-occurs in GNPS/MassIVE datasets.
when_to_use_negative:
- The compounds of interest have no MS/MS spectra — metabolites must be present in
  sufficient quantities or ionize well to be detected and selected for MS/MS acquisition
- The target structures are absent from MASST's ~1.2 billion public spectra, which
  do not encompass all molecules and limit re-analysis of public data
- The goal is structural elucidation of a single unknown rather than repository-scale
  quantification of re-occurring spectra for a synthesized compound panel
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MASST
  role: Produces the repository-scale spectral search job results (USI-keyed match
    records against public metabolomics data) that this skill parses and aggregates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:fe68b57ee8091797b93ab7c5d6e6535e9eff78d24ddb0a4d4203f6b60d53b4e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# masst-job-result-parsing

## Summary

Parses MASST GNPS task result records, keyed per synthesized compound through its USI and MASST job identifier, to aggregate per-compound spectral-match counts and matched public-file counts from repository-scale searches of public metabolomics data.

## When to use

Use when you hold MASST GNPS task result links (with USIs and MASST job identifiers) for MS/MS spectra of synthesized compounds and need per-compound counts of spectral matches and matched public files, e.g. to quantify how widely a synthetic library re-occurs in GNPS/MassIVE datasets.

## When NOT to use

- The compounds of interest have no MS/MS spectra — metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition
- The target structures are absent from MASST's ~1.2 billion public spectra, which do not encompass all molecules and limit re-analysis of public data
- The goal is structural elucidation of a single unknown rather than repository-scale quantification of re-occurring spectra for a synthesized compound panel

## Inputs

- MASST GNPS task result links (per-job match records)
- USI and MASST job identifiers for all synthesized compounds
- Deposited synthetic-mixture MS/MS spectra (MassIVE datasets MSV000089491 for N-acyl amides, MSV000089493 for acyl esters)

## Outputs

- per_compound_match_tables.csv with per-compound spectral-match counts and matched public-file counts
- Per-compound-class aggregates (total matches, matched unique files, fraction of compounds with matches)

## How to apply

Retrieve the deposited synthetic-mixture spectra and the published MASST GNPS task result links via their USI and MASST job identifiers (here MSV000089491 for the N-acyl amide mixture and MSV000089493 for the acyl ester mixture). Parse each MASST job's match records, keyed per synthesized compound through its USI and MASST job identifier. Aggregate matches per compound, counting matched public files and total matches per compound class. Write the per-compound match tables to per_compound_match_tables.csv. Validate by reconciling the compiled totals against reported targets: amides gave 60,277 matches across 25,463 files with 31% of compounds matched, and esters gave 5,884 matches across 5,273 files with 13 esters detected; accept the tables only when totals reconcile.

## Related tools

- **MASST** (Produces the repository-scale spectral search job results (USI-keyed match records against public metabolomics data) that this skill parses and aggregates)

## Examples

```
make run
```

## Evaluation signals

- Amide per-compound totals reconcile to 60,277 spectral matches across 25,463 unique files with 31% of searched compounds matched
- Ester per-compound totals reconcile to 5,884 matches across 5,273 files with 13 esters detected
- Every parsed match record resolves to a synthesized compound via its USI and MASST job identifier
- Matched-file counts per compound never exceed total match counts per compound
- per_compound_match_tables.csv is written and accepted only when totals match the reported figures

## Limitations

- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds; without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data
- MASST's ~1.2 billion spectra do not encompass all molecules, limiting re-analysis of public data
- Detectable matches depend on ionization and abundance — metabolites must ionize well or be present in sufficient quantities to yield MS/MS spectra

## Evidence

- [abstract] Reverse metabolomics workflow: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [supplementary] USI/MASST identifiers: "USI and MASST job identifiers for all synthesized compounds."
- [discussion] MASST coverage limit: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] Isomer false-discovery caveat: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [readme] Workflow test run: "To run the workflow to test simply do"
