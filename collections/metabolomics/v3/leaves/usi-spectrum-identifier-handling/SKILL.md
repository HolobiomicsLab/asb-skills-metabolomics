---
name: usi-spectrum-identifier-handling
description: Use when aggregating MASST matches per synthesized compound — e.g., N-acyl
  amides searched in MSV000089491 or acyl esters in MSV000089493 — where match records
  must be keyed by USI and MASST job identifier.
when_to_use_negative:
- The analysis needs a single one-off spectral library search rather than per-compound
  aggregation of repository-scale MASST results.
- The spectra of interest have no deposited USI or MASST job identifiers (e.g., purely
  local or computationally predicted spectra never searched in public data).
- The goal is definitive isomer-level structure elucidation requiring isolation and
  NMR or X-ray analyses, rather than tracking spectral matches of known synthesized
  compounds.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MASST
  role: Molecular spectrum search executed against public metabolomics repositories;
    each compound's search returns a GNPS task (MASST job) whose match records are
    parsed and keyed by the compound's USI and MASST job identifier.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:32aecfc7bfd6a97aeba2e14d982237864174abd6fe5d063894efddd85de7e3c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASST
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# usi-spectrum-identifier-handling

## Summary

Retrieves and aggregates repository-scale MASST search results for synthesized compound MS/MS spectra, keying every match record by the compound's Universal Spectrum Identifier (USI) and its MASST job identifier, as used in reverse metabolomics to recover spectral matches from public metabolomics data.

## When to use

Use when aggregating MASST matches per synthesized compound — e.g., N-acyl amides searched in MSV000089491 or acyl esters in MSV000089493 — where match records must be keyed by USI and MASST job identifier.

## When NOT to use

- The analysis needs a single one-off spectral library search rather than per-compound aggregation of repository-scale MASST results.
- The spectra of interest have no deposited USI or MASST job identifiers (e.g., purely local or computationally predicted spectra never searched in public data).
- The goal is definitive isomer-level structure elucidation requiring isolation and NMR or X-ray analyses, rather than tracking spectral matches of known synthesized compounds.

## Inputs

- USI strings addressing each synthesized compound's deposited tandem MS/MS spectrum
- MASST job identifiers / GNPS task result links for each compound's repository-scale search
- MassIVE dataset accessions holding the synthetic-mixture spectra (MSV000089491 for N-acyl amides, MSV000089493 for acyl esters)
- Per-job MASST match records listing matched public files and spectra

## Outputs

- per_compound_match_tables.csv with per-compound spectral match counts and matched public file counts
- Per-class aggregates: N-acyl amide targets (60,277 matches, 25,463 files, 31% of compounds matched) and acyl ester targets (5,884 matches, 5,273 files, 13 esters detected)

## How to apply

Each synthesized compound's tandem MS/MS spectrum carries a deposited USI, and the MASST search of that spectrum in public data returns a GNPS task (MASST job identifier); the (USI, MASST job) pair is the unique key for every match record. Retrieve the deposited synthetic-mixture spectra and the published MASST GNPS task result links for the N-acyl amide library under MassIVE dataset MSV000089491 and for the acyl ester library under MSV000089493. Parse each MASST job's match records, keyed per synthesized compound through its USI and MASST job identifier, then aggregate per compound: total matches and count of matched public files, rolled up per compound class. Write the per-compound match tables to per_compound_match_tables.csv. Validate by reconciling compiled totals against the reported figures: amide targets 60,277 matches across 25,463 files with 31% of compounds matched; ester targets 5,884 matches across 5,273 files with 13 esters detected.

## Related tools

- **MASST** (Molecular spectrum search executed against public metabolomics repositories; each compound's search returns a GNPS task (MASST job) whose match records are parsed and keyed by the compound's USI and MASST job identifier.)

## Examples

```
make run
```

## Evaluation signals

- Per-compound match totals and matched-file counts reconcile exactly with reported figures: amides 60,277 matches across 25,463 files (31% of compounds matched); esters 5,884 matches across 5,273 files (13 esters detected).
- Every parsed match record resolves to a unique (USI, MASST job identifier) pair for a synthesized compound; no compound or job is double counted in the aggregation.
- Output per_compound_match_tables.csv is well-formed with one row per synthesized compound carrying total matches and matched public file counts.
- For every compound, matched-file count <= total match count, since one public file can contribute multiple spectral matches.

## Limitations

- MASST's ~1.2 billion public spectra do not encompass all molecules, limiting re-analyses of public data in the discovery phase.
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds; without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded.
- Reverse metabolomics requires an MS/MS spectrum, so compounds not present in sufficient quantities or that ionize poorly cannot be tracked via USI-keyed MASST matches.
- Achievable match coverage depends on public data deposition, particularly with metadata.

## Evidence

- [supplementary] USI and MASST job identifiers for all synthesized compounds.: "USI and MASST job identifiers for all synthesized compounds."
- [abstract] whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [discussion] Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules.: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [readme] You will need to have conda, mamba, and nextflow installed to run things locally.: "You will need to have conda, mamba, and nextflow installed to run things locally."
