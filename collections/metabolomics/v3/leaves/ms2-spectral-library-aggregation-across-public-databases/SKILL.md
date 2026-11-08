---
name: ms2-spectral-library-aggregation-across-public-databases
description: Use when building or auditing a compound-identification spectral library
  that must merge spectra from many public MS2 sources with deduplication and unique-compound
  counting.
when_to_use_negative:
- When processing raw LC–MS or MS/MS data itself — this skill curates reference libraries,
  not spectra from user experiments; use the MetaboAnalyst 6.0 Spectra Processing
  module instead
- When annotating a small targeted panel where an authoritative single-source library
  (e.g. pure HMDB experimental spectra) already covers all analytes
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MetaboAnalyst 6.0
  role: Hosts the resulting curated MS2 reference database and consumes it in its
    MS2 Peak Annotation module for fragment-level compound identification
- name: BUDDY
  role: Supplies predicted fragment formulas for spectra lacking experimentally confirmed
    assignments; predicted entries are tagged separately from measured spectra
- name: R
  role: Implementation language for the aggregation, deduplication and provenance-manifest
    pipeline (MetaboAnalystR 4.0 functions)
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalystR 4.0
  role: R package carrying the knowledgebases (~500,000 metabolite sets) and spectra
    databases (~1.5 million MS2 spectra) for local large-scale processing
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:3840275f047a91eafe0038bdbb9ae14480a791f0bb2537c5da9babb17c8db384
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - BUDDY
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ms2-spectral-library-aggregation-across-public-databases

## Summary

This skill aggregates MS/MS reference spectra from a dozen public databases (HMDB experimental and predicted, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP) plus METLIN neutral-loss spectra into a unified library — 10,420,215 MS2 records over 1,551,012 unique compounds — that underlies MetaboAnalyst 6.0's MS2 Peak Annotation module.

## When to use

Use when building or auditing a compound-identification spectral library that must merge spectra from many public MS2 sources with deduplication and unique-compound counting.

## When NOT to use

- When processing raw LC–MS or MS/MS data itself — this skill curates reference libraries, not spectra from user experiments; use the MetaboAnalyst 6.0 Spectra Processing module instead
- When annotating a small targeted panel where an authoritative single-source library (e.g. pure HMDB experimental spectra) already covers all analytes

## Inputs

- Public MS2 spectral databases: HMDB (experimental + predicted spectra), GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL
- METLIN neutral-loss spectra collection
- Compound metadata (InChIKey, HMDB ID, PubChem CID) from each source

## Outputs

- Curated MS2 reference library with 10,420,215 records covering 1,551,012 unique compounds
- Per-database summary counts and deduplication provenance manifest
- Unified spectral format records with precursor m/z, collision energy, ion mode, and peak lists

## How to apply

Download the latest public releases of the twelve source databases plus METLIN's neutral-loss collection, then standardize each record into a common MS2 spectral format with consistent compound identifiers (InChIKey/HMDB/PubChem) and metadata fields (precursor m/z, collision energy, ion mode, spectrum peaks). Attach BUDDY-predicted fragment formulas to spectra lacking experimentally confirmed formula assignments, tagging predicted entries so they remain distinguishable from measured spectra. Deduplicate spectra across overlapping sources (e.g. MoNA vs GNPS, HMDB vs MassBank) while retaining source provenance, and merge compound-level metadata to compute the unique compound count. Generate per-database summary counts and combined totals, then verify the aggregated library matches the reported 10,420,215 MS2 records and 1,551,012 unique compounds. Finally, serialize the curated library plus a source-provenance manifest so downstream MS2 Peak Annotation modules can consume it.

## Related tools

- **MetaboAnalyst 6.0** (Hosts the resulting curated MS2 reference database and consumes it in its MS2 Peak Annotation module for fragment-level compound identification)
- **BUDDY** (Supplies predicted fragment formulas for spectra lacking experimentally confirmed assignments; predicted entries are tagged separately from measured spectra)
- **R** (Implementation language for the aggregation, deduplication and provenance-manifest pipeline (MetaboAnalystR 4.0 functions)) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalystR 4.0** (R package carrying the knowledgebases (~500,000 metabolite sets) and spectra databases (~1.5 million MS2 spectra) for local large-scale processing) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
Rscript -e 'library(MetaboAnalystR); ms2Library <- LoadMS2Lib("ms2_ref_db.qs"); matched <- MatchMS2(ms2Library, peaks, mz.tol = 25, rt.tol = 0.5)'
```

## Evaluation signals

- The aggregated library must contain exactly 10,420,215 MS2 records covering 1,551,012 unique compounds when reproducing the MetaboAnalyst 6.0 build
- Per-database summary counts should reconcile with the combined total after deduplication (e.g. no double counting between MoNA and GNPS, or HMDB and MassBank)
- Every retained spectrum must carry complete metadata: precursor m/z, collision energy, ion mode, and peak list
- BUDDY-predicted fragment-formula entries must be visibly tagged so they are distinguishable from experimentally measured spectra
- The serialized library plus source-provenance manifest must reproduce identical counts when re-loaded by the MetaboAnalyst 6.0 MS2 Peak Annotation module

## Limitations

- MS2 spectra from DIA data generally require deconvolution before they can be associated with their precursor ions; raw DIA spectra are not directly usable in the library
- MetaboAnalyst's authors state metabolome annotations will continue to improve, implying the aggregated library is an evolving, not final, resource
- The ~1.5 million MS2 spectra shipped with MetaboAnalystR 4.0 is the R-package knowledgebase snapshot and may lag the 10,420,215-record reference described for MetaboAnalyst 6.0

## Evidence

- [other] Reported library size: [evidence span withheld — anchor status: not_found]
- [readme] Spectra databases shipped with MetaboAnalystR: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [abstract] Fragment-level annotation capability: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [readme] DDA/DIA deconvolution need: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [discussion] Future annotation improvements: "we will continue to improve metabolome annotations"
