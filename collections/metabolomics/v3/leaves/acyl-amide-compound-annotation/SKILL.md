---
name: acyl-amide-compound-annotation
description: Use when measuring how widely synthesized N-acyl amide compounds appear
  in public metabolomics repositories — i.e., when MASST GNPS task results, USIs,
  and job identifiers must be parsed and aggregated into per-compound spectral match
  counts.
when_to_use_negative:
- No MS/MS spectrum exists for the target compound — reverse metabolomics requires
  an MS/MS spectrum, and compounds that are too low-abundance or ionize poorly cannot
  be MASST-annotated.
- The goal is definitive isomer-level structure assignment — MASST matching cannot
  exclude closely related isomers without isolation and NMR or X-ray structural analyses.
- Input lacks synthesized reference spectra with USI and MASST job identifiers — matches
  cannot be keyed per compound, so per-compound aggregation is impossible.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MASST
  role: Molecular search engine that queries each synthesized N-acyl amide/ester tandem
    MS/MS spectrum repository-scale against public metabolomics data; its job match
    records, keyed by USI and MASST job identifier, supply every per-compound match
    count
- name: Nextflow
  role: Workflow engine for the Reverse_metabolomics_library_generation pipeline that
    produces and deploys the synthetic-mixture library spectra whose USI/MASST job
    identifiers key this annotation
  repo: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:546c91299a4997eb663ec0635092f6cc8285c7f28265fa26f7ee7512336dcf6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
  tools:
  - MASST
  - Nextflow
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# acyl-amide-compound-annotation

## Summary

Annotates synthesized N-acyl amides repository-scale by MASST-searching each compound's tandem MS/MS spectrum against public metabolomics data and aggregating match records keyed by USI and MASST job identifier into per-compound match tables (amide library: 60,277 matches across 25,463 files; 31% of compounds matched).

## When to use

Use when measuring how widely synthesized N-acyl amide compounds appear in public metabolomics repositories — i.e., when MASST GNPS task results, USIs, and job identifiers must be parsed and aggregated into per-compound spectral match counts.

## When NOT to use

- No MS/MS spectrum exists for the target compound — reverse metabolomics requires an MS/MS spectrum, and compounds that are too low-abundance or ionize poorly cannot be MASST-annotated.
- The goal is definitive isomer-level structure assignment — MASST matching cannot exclude closely related isomers without isolation and NMR or X-ray structural analyses.
- Input lacks synthesized reference spectra with USI and MASST job identifiers — matches cannot be keyed per compound, so per-compound aggregation is impossible.

## Inputs

- Deposited acyl amide synthetic-mixture tandem MS/MS spectra (MassIVE dataset MSV000089491)
- Acyl ester synthetic-mixture spectra (MassIVE dataset MSV000089493)
- USI and MASST job identifiers for all synthesized compounds, with published MASST GNPS task result links
- MASST job match records (per-search match lists keyed by compound USI)

## Outputs

- per_compound_match_tables.csv with per-compound total matches, matched public-file counts, and compound-class aggregates
- Class-level summary metrics (amide: 60,277 matches across 25,463 files, 31% of compounds matched; ester: 5,884 matches across 5,273 files, 13 esters detected)

## How to apply

Retrieve the deposited acyl amide synthetic-mixture tandem MS/MS spectra and their published MASST GNPS task result links for MassIVE dataset MSV000089491, resolving each synthesized compound through its USI and MASST job identifier; repeat for the acyl ester synthetic mixture in MSV000089493. Parse each MASST job's match records, keyed per synthesized compound through its USI and MASST job identifier, so every spectral match maps to exactly one target compound. Aggregate per compound, counting matched public files and total matches per compound class (amide versus ester), and write the per-compound match tables to per_compound_match_tables.csv. Validate by reconciling compiled totals against the reported figures — amide targets: 60,277 matches across 25,463 unique files with 31% of compounds matched; ester targets: 5,884 matches across 5,273 files with 13 esters detected — and accept only when totals match. Rationale: repository-scale reverse metabolomics searches newly synthesized compounds' MS/MS spectra across public data (here ~2,000 unique compounds against MASST's ~1.2 billion spectra) because phenotypic associations emerge from where these structures already occur in deposited human datasets.

## Related tools

- **MASST** (Molecular search engine that queries each synthesized N-acyl amide/ester tandem MS/MS spectrum repository-scale against public metabolomics data; its job match records, keyed by USI and MASST job identifier, supply every per-compound match count)
- **Nextflow** (Workflow engine for the Reverse_metabolomics_library_generation pipeline that produces and deploys the synthetic-mixture library spectra whose USI/MASST job identifiers key this annotation) — https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation

## Examples

```
make run
```

## Evaluation signals

- Compiled amide totals reconcile exactly: 60,277 spectral matches across 25,463 unique files, with 31% of searched amide compounds returning at least one match.
- Ester totals reconcile: 5,884 matches across 5,273 files, with 13 esters detected.
- Every aggregated match row resolves to a unique (USI, MASST job identifier) pair for one synthesized compound.
- For each compound in per_compound_match_tables.csv, the matched-file count is less than or equal to the total match count.
- Re-running the aggregation over the same MASST job records reproduces identical per-compound counts before the tables are accepted.

## Limitations

- MASST's ~1.2 billion public spectra do not encompass all molecules, limiting re-analysis of public data in the discovery phase.
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds; without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded.
- Only 31% of the amide library returned matches because metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition.
- Combinatorial synthesis requires a prior hypothesis about structures of interest, and more complex molecules reduce throughput.

## Evidence

- [abstract] whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
- [supplementary] USI and MASST job identifiers for all synthesized compounds.: "USI and MASST job identifiers for all synthesized compounds."
- [discussion] Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules.: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [discussion] metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition, as reverse metabolomics requires a MS/MS spectrum.: "metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition, as reverse metabolomics requires a MS/MS spectrum."
- [discussion] Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [readme] To run the workflow to test simply do: "To run the workflow to test simply do"
