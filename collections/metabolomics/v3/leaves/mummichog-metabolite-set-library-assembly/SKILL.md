---
name: mummichog-metabolite-set-library-assembly
description: Use when rebuilding, updating, or locally reproducing MetaboAnalyst/MetaboAnalystR
  compound, pathway, or metabolite-set libraries (e.g. after an HMDB release), or
  when verifying reported library scale metrics.
when_to_use_negative:
- Input is already a processed feature table or metabolite intensity matrix — library
  assembly operates on source knowledgebases, not user data
- Only one organism's pathway library is needed for a single analysis run and the
  shipped .qs libraries are current — rebuild is unnecessary
- The analysis requires MS2 spectral library building rather than metabolite-set/pathway
  knowledgebases
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetaboAnalystR 4.0
  role: Provides the R functions (.readDataTable, CrossReferencing, CreateMummichogLibs,
    GetMsetLibCheckMsg) used to parse, rebuild, validate, and serialize the knowledgebase
    libraries
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose updated compound database (from HMDB 5.0), pathway libraries
    (~130 species), and metabolite set libraries are reproduced by this skill
- name: R
  role: Runtime environment for MetaboAnalystR library-rebuild scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:6e9c62dd4ec8f756fe0ed8fafd566ffd76acb800ac0a2de3b61342dcee6c5ceb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - .readDataTable
  - CreateMummichogLibs
  - CrossReferencing
  - GetMsetLibCheckMsg
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# mummichog-metabolite-set-library-assembly

## Summary

Assembles and validates the knowledge libraries (compound database, KEGG pathway libraries, metabolite set libraries) that underpin mummichog-style pathway analysis in MetaboAnalyst 6.0/MetaboAnalystR 4.0, by rebuilding them from HMDB 5.0, KEGG API, MarkerDB, and RaMP-DB sources and serializing them as .qs files.

## When to use

Use when rebuilding, updating, or locally reproducing MetaboAnalyst/MetaboAnalystR compound, pathway, or metabolite-set libraries (e.g. after an HMDB release), or when verifying reported library scale metrics.

## When NOT to use

- Input is already a processed feature table or metabolite intensity matrix — library assembly operates on source knowledgebases, not user data
- Only one organism's pathway library is needed for a single analysis run and the shipped .qs libraries are current — rebuild is unnecessary
- The analysis requires MS2 spectral library building rather than metabolite-set/pathway knowledgebases

## Inputs

- HMDB 5.0 metabolite definitions (XML download)
- KEGG REST API responses for compound and pathway membership per organism
- MarkerDB metabolite sets
- RaMP-DB pathway definitions (~3700 pathways)
- Existing MetaboAnalystR knowledgebase folder to be rebuilt

## Outputs

- Rebuilt compound database with synchronized KEGG/PubChem/ChEBI/METLIN/LIPID Maps IDs
- KEGG pathway libraries serialized as .qs files per supported organism
- Metabolite set libraries serialized as .qs files
- knowledgebase_summary.json reporting net compound expansion, species count, and RaMP-DB pathway count

## How to apply

Download HMDB 5.0 metabolite definitions and parse them into a compound table keyed by HMDB accession using .readDataTable. Synchronize cross-database identifiers (KEGG, PubChem, ChEBI, METLIN, LIPID Maps) for each compound via the CrossReferencing routine with q.type='mixed'. Query the KEGG REST API for compound and pathway membership of each supported organism and rebuild the pathway libraries with CreateMummichogLibs(folder, kegg_compounds); the KEGG library refresh is dated 12/20/2023 and pathway analysis supports ~130 species (up from 28). Regenerate metabolite set libraries from HMDB 5.0, MarkerDB, and RaMP-DB sources, serializing as .qs files (replacing the older .rds format), incorporating ~3700 RaMP-DB pathways and 62 dietary/chemical-exposure metabolite sets. The net compound expansion vs. HMDB 4.0-era libraries is ~4,000 entries (~10,000 deprecated HMDB entries removed, ~14,000 added). Run GetMsetLibCheckMsg on each rebuilt library to confirm internal consistency of compound-to-set mappings, then write knowledgebase_summary.json with the verified counts.

## Related tools

- **MetaboAnalystR 4.0** (Provides the R functions (.readDataTable, CrossReferencing, CreateMummichogLibs, GetMsetLibCheckMsg) used to parse, rebuild, validate, and serialize the knowledgebase libraries) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose updated compound database (from HMDB 5.0), pathway libraries (~130 species), and metabolite set libraries are reproduced by this skill)
- **R** (Runtime environment for MetaboAnalystR library-rebuild scripts)

## Examples

```
lib <- CreateMummichogLibs(folder, kegg_compounds); msg <- GetMsetLibCheckMsg(lib)
```

## Evaluation signals

- knowledgebase_summary.json reports a net compound expansion of ~4,000 entries (≈10,000 deprecated HMDB entries removed, ≈14,000 added)
- Rebuilt KEGG pathway libraries cover ~130 supported species (up from 28)
- Metabolite set libraries contain ~3,700 RaMP-DB pathways and 62 dietary/chemical-exposure metabolite sets
- GetMsetLibCheckMsg returns no inconsistency for compound-to-set mappings in every rebuilt library
- All serialized libraries use the .qs format rather than the legacy .rds format

## Limitations

- The KEGG pathway library refresh is pinned to a 12/20/2023 KEGG API snapshot, so downstream pathway membership reflects that release
- Compounds removed in HMDB 5.0 (~10,000 deprecated entries) are no longer resolvable, which can break mappings against legacy feature lists
- Legacy consumers expecting .rds library files are incompatible with the new .qs serialization

## Evidence

- [abstract] Pathway-analysis species expansion: "significantly expanded its pathway analysis support to around 130 species"
- [methods] MetaboAnalystR function/library basis: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] Knowledgebase scale shipped with MetaboAnalystR: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
