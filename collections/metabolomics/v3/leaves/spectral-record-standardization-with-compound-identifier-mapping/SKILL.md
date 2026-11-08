---
name: spectral-record-standardization-with-compound-identifier-mapping
description: Use when raw MS2 spectra records from multiple public databases must
  be merged into one standardized, identifier-mapped, deduplicated reference library
  for compound identification.
when_to_use_negative:
- Input is already a standardized spectral library (e.g. the shipped MetaboAnalystR
  knowledgebases) — query it instead of rebuilding
- The task is MS1 peak detection/feature quantification (Spectra Processing with asari/OptiLCMS),
  not MS2 reference-library curation
- Only a handful of spectra need annotation against a single database without cross-source
  deduplication or identifier mapping
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR
  role: Ships the spectra databases (~1.5 million MS2 spectra in v4.0) and R functions
    underlying the MetaboAnalyst web server; the curated library feeds its MS2 Peak
    Annotation module
  repo: https://github.com/xia-lab/MetaboAnalystR.git
- name: BUDDY
  role: Predicts fragment formulas for MS2 spectra lacking experimentally confirmed
    formula assignments; predicted entries are tagged to stay distinguishable from
    measured spectra
- name: R
  role: Base environment (version > 3.6.1 recommended) in which the standardization,
    deduplication, and aggregation pipeline is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:2a6732415f19dbe10156cfa7e7114bd7e68bc4bf661682e2fc0354457f58a5f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR.git
  tools:
  - BUDDY
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-record-standardization-with-compound-identifier-mapping

## Summary

Builds a curated MS2 reference spectral library by harmonizing records from heterogeneous public databases (HMDB, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL, METLIN) into a common spectral format with unified compound identifiers, underpinning the MetaboAnalyst 6.0 MS2 Peak Annotation module.

## When to use

Use when raw MS2 spectra records from multiple public databases must be merged into one standardized, identifier-mapped, deduplicated reference library for compound identification.

## When NOT to use

- Input is already a standardized spectral library (e.g. the shipped MetaboAnalystR knowledgebases) — query it instead of rebuilding
- The task is MS1 peak detection/feature quantification (Spectra Processing with asari/OptiLCMS), not MS2 reference-library curation
- Only a handful of spectra need annotation against a single database without cross-source deduplication or identifier mapping

## Inputs

- Latest public MS2 spectral releases from HMDB (experimental and predicted), GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, and MS-DIAL
- METLIN neutral-loss spectra collection
- Compound metadata records carrying identifiers (InChIKey, HMDB, PubChem) and precursor m/z, collision energy, ion mode, spectrum peaks fields
- BUDDY for fragment formula prediction on spectra lacking confirmed assignments

## Outputs

- Standardized, deduplicated MS2 spectral library (10,420,215 records; 1,551,012 unique compounds) with unified compound identifiers
- Per-database summary counts and combined aggregate totals
- Source-provenance manifest linking each retained spectrum to its originating database
- Curated library serialized for use by the MetaboAnalyst 6.0 MS2 Peak Annotation module

## How to apply

Download the latest public releases of the 12 MS2 sources plus the METLIN neutral-loss collection, then standardize every record into a common MS2 spectral format carrying precursor m/z, collision energy, ion mode, and spectrum peaks. Normalize compound identifiers across sources to a consistent key set (InChIKey, HMDB, PubChem) so that the same compound contributed by different databases merges cleanly. For spectra lacking experimentally confirmed formula assignments, attach BUDDY-predicted fragment formulas and tag these entries so predicted spectra remain distinguishable from measured ones. Deduplicate overlapping records (e.g. MoNA vs GNPS, HMDB vs MassBank) while retaining source provenance, merge compound-level metadata, and compute per-database and combined counts; the aggregated library should match 10,420,215 MS2 records covering 1,551,012 unique compounds. Finally serialize the curated library plus a source-provenance manifest for consumption by the MS2 Peak Annotation module.

## Related tools

- **MetaboAnalystR** (Ships the spectra databases (~1.5 million MS2 spectra in v4.0) and R functions underlying the MetaboAnalyst web server; the curated library feeds its MS2 Peak Annotation module) — https://github.com/xia-lab/MetaboAnalystR.git
- **BUDDY** (Predicts fragment formulas for MS2 spectra lacking experimentally confirmed formula assignments; predicted entries are tagged to stay distinguishable from measured spectra)
- **R** (Base environment (version > 3.6.1 recommended) in which the standardization, deduplication, and aggregation pipeline is executed)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Combined record total equals 10,420,215 MS2 spectra and unique compound count equals 1,551,012 after deduplication
- Every retained record carries at least one consistent compound identifier (InChIKey, HMDB, or PubChem) plus precursor m/z, collision energy, ion mode, and peak list
- BUDDY-predicted fragment-formula entries are explicitly tagged and separable from experimentally measured spectra
- Per-database summary counts sum to the combined total, and overlapping duplicates (MoNA vs GNPS, HMDB vs MassBank) are removed while provenance is preserved
- Library loads successfully into the MetaboAnalyst 6.0 MS2 Peak Annotation module for fragment-level annotation

## Limitations

- Spectra augmented with BUDDY-predicted fragment formulas are computational predictions, not experimentally confirmed assignments, and must remain tagged as such
- Heavy source overlap (e.g. MoNA vs GNPS, HMDB vs MassBank) makes deduplication error-prone if provenance is not retained through the merge
- Version drift: the MetaboAnalystR 4.0 README advertises ~1.5 million MS2 spectra, whereas the MetaboAnalyst 6.0 curated build aggregates 10,420,215 records from 12+ sources — counts are version- and build-specific

## Evidence

- [abstract] a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [readme] a large collection of knowledgebases and spectra databases: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [intro] MS2 spectra from pooled QC samples assist compound identification: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
