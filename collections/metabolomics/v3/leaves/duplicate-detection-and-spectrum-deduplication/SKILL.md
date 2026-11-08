---
name: duplicate-detection-and-spectrum-deduplication
description: Use when combining several public MS/MS spectral databases into one curated
  reference library and you must remove cross-source duplicates (e.g. MoNA vs GNPS,
  HMDB vs MassBank) without losing provenance.
when_to_use_negative:
- You are deconvolving sample-acquired DIA MS2 spectra to reconstruct precursor associations
  — that is spectra processing, not reference-library curation
- The input is already a single curated library with no overlapping source releases
  to merge
- You need compound identification scoring against a finished library rather than
  building or maintaining the library itself
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalyst 6.0
  role: Consumes the deduplicated curated MS2 library in its MS2 Peak Annotation module;
    the curation workflow reproduces its reference database build
- name: BUDDY
  role: Attaches predicted fragment formulas to spectra lacking experimentally confirmed
    formula assignments, with predicted entries tagged
- name: R / MetaboAnalystR
  role: Implementation environment for standardizing records, deduplicating, and serializing
    the curated library; MetaboAnalystR ships the packaged spectra databases
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:151b5f0af4cd6b40a263b34430aa11878d242c8fb01f182ab7981b6dcd0508b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - BUDDY
  - MetaboAnalyst 6.0
  - R
  - R / MetaboAnalystR
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Duplicate detection and spectrum deduplication

## Summary

Aggregates multiple public MS2 reference libraries (HMDB, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL, METLIN) into a single curated spectral database, deduplicating spectra that overlap across sources while retaining source provenance and merging compound metadata to compute unique compound counts.

## When to use

Use when combining several public MS/MS spectral databases into one curated reference library and you must remove cross-source duplicates (e.g. MoNA vs GNPS, HMDB vs MassBank) without losing provenance.

## When NOT to use

- You are deconvolving sample-acquired DIA MS2 spectra to reconstruct precursor associations — that is spectra processing, not reference-library curation
- The input is already a single curated library with no overlapping source releases to merge
- You need compound identification scoring against a finished library rather than building or maintaining the library itself

## Inputs

- Public MS2 reference database releases: HMDB (experimental and predicted spectra), GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL
- METLIN neutral-loss spectra collection
- Per-record metadata: precursor m/z, collision energy, ion mode, spectrum peaks, compound identifiers (InChIKey/HMDB/PubChem)

## Outputs

- Deduplicated curated MS2 spectral library in a standardized format with per-record source provenance
- BUDDY-predicted fragment formula annotations, tagged as predicted rather than measured
- Per-database summary counts and combined totals (MS2 records and unique compounds)
- Source-provenance manifest for use by the MetaboAnalyst 6.0 MS2 Peak Annotation module

## How to apply

First standardize every source's records into a common MS2 spectral format with consistent compound identifiers (InChIKey/HMDB/PubChem) and metadata fields: precursor m/z, collision energy, ion mode, and spectrum peaks. Spectra lacking experimentally confirmed formula assignments are annotated with BUDDY-predicted fragment formulas and tagged so predicted entries remain distinguishable from measured spectra. Then detect duplicates across overlapping sources (e.g. MoNA vs GNPS, HMDB vs MassBank) and collapse them while retaining per-source provenance, and merge compound-level metadata to compute the unique compound count. Generate per-database summary counts and combined totals, and verify the aggregated library against the reported totals (10,420,215 MS2 records; 1,551,012 unique compounds) before serializing the curated library plus a source-provenance manifest for the MetaboAnalyst 6.0 MS2 Peak Annotation module.

## Related tools

- **MetaboAnalyst 6.0** (Consumes the deduplicated curated MS2 library in its MS2 Peak Annotation module; the curation workflow reproduces its reference database build)
- **BUDDY** (Attaches predicted fragment formulas to spectra lacking experimentally confirmed formula assignments, with predicted entries tagged)
- **R / MetaboAnalystR** (Implementation environment for standardizing records, deduplicating, and serializing the curated library; MetaboAnalystR ships the packaged spectra databases) — https://github.com/xia-lab/MetaboAnalystR

## Evaluation signals

- Aggregated MS2 record count matches the reported total of 10,420,215
- Unique compound count (after deduplication and metadata merge) matches 1,551,012
- No identical spectra remain duplicated across overlapping source pairs (e.g. MoNA vs GNPS, HMDB vs MassBank), while each retained record still carries source provenance
- Predicted (BUDDY-formula) entries remain distinguishable from experimentally measured spectra throughout the library
- Per-database summary counts reconcile with the combined total after deduplication

## Limitations

- The packaged MetaboAnalystR 4.0 distribution bundles ~1.5 million MS2 spectra, a different scope/snapshot from the full 10.4M-record aggregated library described for MetaboAnalyst 6.0
- Deduplication depends on consistent compound identifiers across sources; inconsistencies in InChIKey/HMDB/PubChem mapping can inflate or deflate the unique compound count
- Spectra with BUDDY-predicted (rather than experimentally confirmed) fragment formulas must stay tagged, or predicted and measured spectra will be conflated

## Evidence

- [abstract] Task-card workflow support: the article describes the MS2 Peak Annotation module built on comprehensive reference databases: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [methods] Scale of the spectra databases bundled with the R implementation: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] README confirmation that MetaboAnalystR ships spectra databases to support local large-scale processing: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra) to support local large-scale processing"
- [intro] Rationale for maintaining a comprehensive, deduplicated MS2 reference library: "Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation."
