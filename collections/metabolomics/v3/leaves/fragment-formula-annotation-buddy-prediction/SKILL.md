---
name: fragment-formula-annotation-buddy-prediction
description: Use when building or extending an MS2 reference spectral library for
  compound identification and source spectra lack confirmed fragment formula assignments.
when_to_use_negative:
- Spectra already carry experimentally confirmed fragment formula assignments and
  only need deduplication or provenance merging
- The task is MS1 feature detection or peak picking rather than MS2 spectral library
  curation and annotation
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalyst 6.0
  role: Host platform whose MS2 Peak Annotation module consumes the curated MS2 reference
    database with fragment-level annotation
- name: BUDDY
  role: Predicts fragment formulas for MS2 spectra lacking experimentally confirmed
    formula assignments
- name: R
  role: Scripting environment for standardizing, deduplicating, and serializing the
    curated library (R base > 3.6.1 recommended)
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalystR 4.0
  role: R package bundling the spectra databases (~1.5 million MS2 spectra) for local
    large-scale processing or API service
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:3680da8ccbe733498d5437c376d418fc6a31fa4bc0c58dc2cf4c3a77191bac7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - BUDDY
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fragment-formula-annotation-buddy-prediction

## Summary

Attaches BUDDY-predicted fragment formulas to MS2 reference spectra that lack experimentally confirmed formula assignments, expanding the curated MS2 library (10,420,215 records over 1,551,012 unique compounds) that underpins MetaboAnalyst 6.0's MS2 Peak Annotation module.

## When to use

Use when building or extending an MS2 reference spectral library for compound identification and source spectra lack confirmed fragment formula assignments.

## When NOT to use

- Spectra already carry experimentally confirmed fragment formula assignments and only need deduplication or provenance merging
- The task is MS1 feature detection or peak picking rather than MS2 spectral library curation and annotation

## Inputs

- Public MS2 reference database releases (HMDB experimental and predicted spectra, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL)
- METLIN neutral-loss spectra collection
- MS2 spectral records with precursor m/z, collision energy, ion mode, and spectrum peaks
- Compound identifiers (InChIKey/HMDB/PubChem)

## Outputs

- Curated MS2 spectral library with BUDDY-predicted fragment formulas tagged as predicted vs measured
- Source-provenance manifest for the curated library
- Per-database summary counts and combined totals (10,420,215 MS2 records; 1,551,012 unique compounds)

## How to apply

Collect MS2 spectra from the 12 public sources (HMDB experimental and predicted, GNPS, MoNA, MassBank, MINEs, LipidBlast, RIKEN, ReSpect, BMDMS, VaniyaNP, MS-DIAL) plus METLIN neutral-loss spectra, and standardize them with consistent compound identifiers (InChIKey/HMDB/PubChem) and metadata fields (precursor m/z, collision energy, ion mode, spectrum peaks). Identify spectra without experimentally confirmed fragment formula assignments and run BUDDY to predict fragment formulas for those entries, tagging them as predicted so they remain distinguishable from measured spectra. Deduplicate overlapping sources (e.g. MoNA vs GNPS, HMDB vs MassBank) while retaining source provenance, merge compound-level metadata, and compute per-database and combined counts. Serialize the curated library with a source-provenance manifest for use by the MetaboAnalyst 6.0 MS2 Peak Annotation module, which performs fragment-level annotation against these comprehensive MS2 reference databases.

## Related tools

- **MetaboAnalyst 6.0** (Host platform whose MS2 Peak Annotation module consumes the curated MS2 reference database with fragment-level annotation)
- **BUDDY** (Predicts fragment formulas for MS2 spectra lacking experimentally confirmed formula assignments)
- **R** (Scripting environment for standardizing, deduplicating, and serializing the curated library (R base > 3.6.1 recommended)) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalystR 4.0** (R package bundling the spectra databases (~1.5 million MS2 spectra) for local large-scale processing or API service) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual = T)
```

## Evaluation signals

- Aggregated library matches the reported totals of 10,420,215 MS2 records and 1,551,012 unique compounds
- Every spectrum lacking an experimental formula carries a BUDDY-predicted formula entry explicitly tagged as predicted
- Deduplication across overlapping sources (MoNA vs GNPS, HMDB vs MassBank) preserves source provenance without double counting
- Compound identifiers resolve consistently to InChIKey/HMDB/PubChem so unique compound counts are reproducible
- Serialized library loads in the MetaboAnalyst 6.0 MS2 Peak Annotation module and supports fragment-level annotation

## Limitations

- BUDDY-assigned formulas are predictions, not measurements; predicted entries must remain distinguishable from measured spectra
- The article notes metabolome annotations still need improvement and will continue to be improved (stated as future work)
- The ~1.5 million MS2 spectra bundled with MetaboAnalystR is an approximation of the full curated library

## Evidence

- [abstract] MS2 peak annotation with fragment-level annotation against MS2 reference databases: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [methods] a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra): "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [methods] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] spectra databases (~1.5 million MS2 spectra) to support local large-scale processing or using our API service: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra) to support local large-scale processing or"
- [discussion] we will continue to improve metabolome annotations: "we will continue to improve metabolome annotations"
