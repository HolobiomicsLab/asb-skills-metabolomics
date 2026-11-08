---
name: database-count-verification-and-provenance-tracking
description: Use when a paper reports an aggregate count for a curated reference database
  (records and unique entities) and you must reproduce the build, reconcile the total,
  and retain source provenance.
when_to_use_negative:
- Your input is already a merged, deduplicated feature or annotation table — counting
  rows there verifies nothing about the underlying database build
- You only need to run compound identification against an existing reference library
  (use the MS2 Peak Annotation module directly rather than rebuilding the database)
- The claim you are verifying concerns a single-source database with no overlapping/duplicate
  entries, where a simple row count suffices
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboAnalyst 6.0
  role: Target platform whose curated MS2 reference database is being reproduced and
    verified for the MS2 Peak Annotation module
- name: BUDDY
  role: Predicts fragment formulas for spectra lacking experimentally confirmed assignments;
    predicted entries are tagged to stay distinguishable from measured spectra
- name: MetaboAnalystR
  role: R package bundling the spectra databases for local verification and large-scale
    processing of the MS2 library
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Implementation environment (R base > 3.6.1 recommended) for standardizing,
    deduplicating, and tallying the library
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:cecce728b7fb0aa7c485b29ed50ee54c63c4059235029e88cb1f9bf2b56fe729
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - BUDDY
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# database-count-verification-and-provenance-tracking

## Summary

Reproduce and verify the size of a merged multi-source reference database (e.g. MetaboAnalyst 6.0's curated MS2 spectral library: 10,420,215 MS2 records over 1,551,012 unique compounds) while tracking per-source provenance and deduplication decisions.

## When to use

Use when a paper reports an aggregate count for a curated reference database (records and unique entities) and you must reproduce the build, reconcile the total, and retain source provenance.

## When NOT to use

- Your input is already a merged, deduplicated feature or annotation table — counting rows there verifies nothing about the underlying database build
- You only need to run compound identification against an existing reference library (use the MS2 Peak Annotation module directly rather than rebuilding the database)
- The claim you are verifying concerns a single-source database with no overlapping/duplicate entries, where a simple row count suffices

## Inputs

- Public releases of the 12 MS2 reference databases (HMDB experimental/predicted, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL)
- METLIN neutral-loss spectra collection
- BUDDY-predicted fragment formula assignments for spectra lacking confirmed formulas

## Outputs

- Curated MS2 spectral library in a standardized common format with consistent compound identifiers (InChIKey/HMDB/PubChem)
- Per-database summary counts and combined totals
- Source-provenance manifest with deduplication and predicted-vs-measured tagging

## How to apply

Download the latest public releases of each contributing source (for MetaboAnalyst 6.0's MS2 library: HMDB experimental and predicted spectra, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL, plus the METLIN neutral-loss collection). Standardize each source into a common MS2 spectral format with consistent compound identifiers (InChIKey/HMDB/PubChem) and metadata fields (precursor m/z, collision energy, ion mode, spectrum peaks); attach BUDDY-predicted fragment formulas to spectra lacking experimentally confirmed assignments, tagging predicted entries so they remain distinguishable from measured spectra. Deduplicate overlapping sources (e.g. MoNA vs GNPS, HMDB vs MassBank) while retaining source provenance, then merge compound-level metadata to compute the unique compound count. Generate per-database summary counts and combined totals, and judge success by whether the aggregated library matches the reported totals (10,420,215 MS2 records and 1,551,012 unique compounds) before serializing the library plus a provenance manifest for downstream annotation modules.

## Related tools

- **MetaboAnalyst 6.0** (Target platform whose curated MS2 reference database is being reproduced and verified for the MS2 Peak Annotation module)
- **BUDDY** (Predicts fragment formulas for spectra lacking experimentally confirmed assignments; predicted entries are tagged to stay distinguishable from measured spectra)
- **MetaboAnalystR** (R package bundling the spectra databases for local verification and large-scale processing of the MS2 library) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Implementation environment (R base > 3.6.1 recommended) for standardizing, deduplicating, and tallying the library)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual = T)
```

## Evaluation signals

- Aggregated library total equals the reported 10,420,215 MS2 records
- Unique compound count after deduplication and metadata merge equals 1,551,012
- Per-database summary counts reconcile exactly with the combined total (no records lost or double-counted across e.g. MoNA vs GNPS, HMDB vs MassBank)
- Every record in the serialized library carries source provenance, and BUDDY-predicted entries remain tagged separately from measured spectra
- Library loads successfully into the MS2 Peak Annotation module as a usable reference database

## Limitations

- The README for MetaboAnalystR 4.0 reports '~1.5 million MS2 spectra' while the MetaboAnalyst 6.0 build totals ~10.4 million records — version and packaging drift between the R package knowledgebases and the web platform's curated database must be accounted for when reconciling counts
- Spectra lacking experimentally confirmed formula assignments rely on BUDDY predictions and must stay tagged; conflating predicted with measured spectra inflates the experimentally measured record count
- Overlapping sources (MoNA vs GNPS, HMDB vs MassBank) make deduplication policy a material decision point that changes the final totals

## Evidence

- [methods] a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra): "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra): "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [abstract] a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
