---
name: kegg-pathway-library-construction
description: Use when rebuilding or refreshing metabolomics pathway-analysis libraries
  from KEGG, or when verifying reported library counts (e.g. ~130 species, ~3700 RaMP-DB
  pathways) after a knowledgebase update.
when_to_use_negative:
- Input is already a feature table of quantified peaks — pathway-library construction
  is a knowledgebase maintenance task, not a data-analysis step
- The analysis targets a species not supported by the KEGG pathway libraries (~130
  species supported); pathway analysis will not apply
- Only MS2 spectra annotation is needed — use the MS2 Peak Annotation module with
  the spectra databases (~1.5 million MS2 spectra) instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalyst 6.0
  role: Platform whose updated compound database, pathway libraries, and metabolite
    set libraries are reproduced by this skill; pathway analysis expanded to ~130
    species
- name: MetaboAnalystR 4.0
  role: R package providing the underlying functions (.readDataTable, CrossReferencing,
    CreateMummichogLibs, GetMsetLibCheckMsg) and knowledgebases used to rebuild the
    libraries
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Runtime environment for executing the library-rebuild workflow (R base > 3.6.1
    recommended)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:4a422f8445ffc21c52f81cb54ccd1e24310922a3bc15fd984692a421c8c10f36
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# kegg-pathway-library-construction

## Summary

Rebuild MetaboAnalyst's KEGG pathway libraries by querying the KEGG REST API for compound and pathway membership across supported organisms, then validating compound-to-pathway mappings — this underpins pathway analysis expansion from 28 to ~130 species in MetaboAnalyst 6.0.

## When to use

Use when rebuilding or refreshing metabolomics pathway-analysis libraries from KEGG, or when verifying reported library counts (e.g. ~130 species, ~3700 RaMP-DB pathways) after a knowledgebase update.

## When NOT to use

- Input is already a feature table of quantified peaks — pathway-library construction is a knowledgebase maintenance task, not a data-analysis step
- The analysis targets a species not supported by the KEGG pathway libraries (~130 species supported); pathway analysis will not apply
- Only MS2 spectra annotation is needed — use the MS2 Peak Annotation module with the spectra databases (~1.5 million MS2 spectra) instead

## Inputs

- HMDB 5.0 metabolite definitions
- KEGG REST API responses (compound and pathway membership per organism)
- Cross-referenced compound table with KEGG and PubChem IDs keyed by HMDB accession

## Outputs

- Rebuilt KEGG pathway libraries serialized as .qs files
- Metabolite set libraries based on HMDB 5.0, MarkerDB, and RaMP-DB (~3700 pathways, 62 exposure-associated metabolite sets)
- knowledgebase_summary.json reporting validated counts

## How to apply

Synchronize cross-database compound identifiers (KEGG, PubChem, ChEBI, METLIN, LIPID Maps) using the CrossReferencing routine with q.type='mixed', starting from the HMDB 5.0 metabolite definitions parsed with .readDataTable and keyed by HMDB accession. Query the KEGG REST API for compound and pathway membership of each supported organism, then rebuild the pathway libraries with CreateMummichogLibs(folder, kegg_compounds). Serialize the rebuilt libraries as .qs files (replacing the older .rds format). Run GetMsetLibCheckMsg on each rebuilt library to confirm internal consistency of compound-to-set mappings, and check the reported scale metrics: net compound expansion of ~4,000 entries (removing ~10,000 deprecated HMDB entries and adding ~14,000 new ones) and pathway-analysis support for ~130 species (up from 28).

## Related tools

- **MetaboAnalyst 6.0** (Platform whose updated compound database, pathway libraries, and metabolite set libraries are reproduced by this skill; pathway analysis expanded to ~130 species)
- **MetaboAnalystR 4.0** (R package providing the underlying functions (.readDataTable, CrossReferencing, CreateMummichogLibs, GetMsetLibCheckMsg) and knowledgebases used to rebuild the libraries) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Runtime environment for executing the library-rebuild workflow (R base > 3.6.1 recommended))

## Examples

```
CreateMummichogLibs(folder, kegg_compounds); GetMsetLibCheckMsg(mSetObj)
```

## Evaluation signals

- knowledgebase_summary.json reports a net compound expansion of ~4,000 entries (~14,000 new HMDB entries added, ~10,000 deprecated removed)
- Rebuilt pathway libraries support pathway analysis for ~130 species (up from 28)
- Metabolite set libraries incorporate ~3,700 RaMP-DB pathways and 62 metabolite sets associated with dietary and chemical exposures
- GetMsetLibCheckMsg reports no internal-consistency errors for compound-to-set mappings in each rebuilt library
- All library files are serialized in .qs format (not the older .rds format)

## Limitations

- KEGG pathway libraries are refreshed via the KEGG API on a specific date (e.g. 12/20/2023 snapshot); downstream results depend on the KEGG release at build time
- Metabolome annotations still need improvement, as acknowledged as future work by the authors
- Compound database synchronization depends on HMDB 5.0, including removal of ~10,000 deprecated HMDB entries, which may break lookups of legacy HMDB accessions

## Evidence

- [abstract] Pathway analysis support expanded to around 130 species, with improved visualization, updated compound database and metabolite sets: "significantly expanded its pathway analysis support to around 130 species"
- [methods] MetaboAnalystR knowledgebases scale: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [methods] MetaboAnalystR package role: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] MetaboAnalystR is synchronized with the web server for reproducibility: "The package is synchronized with the MetaboAnalyst web server. After installing and loading the package, users will be able to reproduce the same results from their local computers"
- [discussion] Future-work limitations on annotations and omics integration: "we will continue to improve metabolome annotations"
