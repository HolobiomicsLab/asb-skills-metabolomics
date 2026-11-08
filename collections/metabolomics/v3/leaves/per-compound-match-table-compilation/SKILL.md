---
name: per-compound-match-table-compilation
description: Use when you have parsed MASST match records for a library of synthesized
  compounds and need to compile per-compound counts of matched public files and total
  matches, then reconcile totals against reported figures.
when_to_use_negative:
- You are running MASST searches themselves rather than compiling/aggregating already-produced
  MASST job results.
- Your goal is compound discovery or structural annotation rather than quantifying
  repository-wide spectral match recovery for a known library of synthesized compounds.
- Compounds lack deposited spectra with USIs and MASST job identifiers, so per-compound
  match records cannot be keyed.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MASST
  role: Mass spectrometry search tool whose job results provide the per-compound spectral
    match records aggregated into the tables; job identifiers key records to synthesized
    compounds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:836996de5da63b3ca9b512b05c7b3bac69010919307ee05c0ea104f7f10f5bba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# per-compound-match-table-compilation

## Summary

Aggregates MASST search results for synthesized compounds (keyed by USI and MASST job identifier) into per-compound match tables, counting matched public files and total matches per compound class, as done in the reverse metabolomics workflow for N-acyl amides and acyl esters.

## When to use

Use when you have parsed MASST match records for a library of synthesized compounds and need to compile per-compound counts of matched public files and total matches, then reconcile totals against reported figures.

## When NOT to use

- You are running MASST searches themselves rather than compiling/aggregating already-produced MASST job results.
- Your goal is compound discovery or structural annotation rather than quantifying repository-wide spectral match recovery for a known library of synthesized compounds.
- Compounds lack deposited spectra with USIs and MASST job identifiers, so per-compound match records cannot be keyed.

## Inputs

- MASST GNPS task result links/match records for each synthesized compound (e.g. datasets MSV000089491 and MSV000089493)
- Universal Spectrum Identifiers (USIs) and MASST job identifiers for all synthesized compounds
- Deposited acyl amide and acyl ester synthetic-mixture spectra

## Outputs

- per_compound_match_tables.csv with per-compound counts of matched public files and total matches, per compound class (amide targets: 60,277 matches across 25,463 files, 31% of compounds matched; ester targets: 5,884 matches across 5,273 files, 13 esters detected)

## How to apply

Retrieve the deposited synthetic-mixture spectra via their USIs and the published MASST GNPS task result links for each dataset (e.g. amide targets from MSV000089491 and ester targets from MSV000089493). Parse each MASST job's match records, keying each record to its synthesized compound through the USI and MASST job identifier. Aggregate matches per compound, counting matched public files and total matches per compound class (e.g. amide vs. ester targets). Write the aggregated tables to per_compound_match_tables.csv. Validate by reconciling compiled totals and matched-file counts against reported values, accepting the tables only when the totals match the reported figures.

## Related tools

- **MASST** (Mass spectrometry search tool whose job results provide the per-compound spectral match records aggregated into the tables; job identifiers key records to synthesized compounds)

## Examples

```
make run
```

## Evaluation signals

- Compiled amide-target totals reconcile to the reported values: 60,277 spectral matches across 25,463 unique files, with 31% of searched compounds returning spectral matches.
- Compiled ester-target totals reconcile to the reported values: 5,884 matches across 5,273 files, with 13 esters detected.
- Every match record in the tables resolves to a synthesized compound via its USI and MASST job identifier.
- Matched-file counts are de-duplicated per compound (unique files) rather than raw match counts.
- per_compound_match_tables.csv is produced with per-compound and per-class aggregates.

## Limitations

- The 1.2 billion spectra currently part of MASST searches do not encompass all molecules, limiting re-analysis of public data in the discovery phase.
- MS/MS matching does not guarantee against false discovery of closely related isomers or compounds; without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded.
- Only compounds with adequate abundance/ionization yielding an MS/MS spectrum can be searched; reverse metabolomics requires an MS/MS spectrum.

## Evidence

- [supplementary] USI and MASST job identifiers for all synthesized compounds.: "USI and MASST job identifiers for all synthesized compounds."
- [abstract] Reverse metabolomics searches spectra in public data: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [discussion] MASST coverage limitation: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] Isomer ambiguity caveat: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [intro] Nextflow workflow execution: "To run the workflow to test simply do"
