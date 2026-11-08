---
name: sqlite-annotation-results-consolidation
description: Use when multiple MI-Pack annotation searches have written SQLite output
  files and a consolidated summary file of metabolite annotations is needed, or when
  a single SQLite output needs to be summarized directly.
when_to_use_negative:
- Peak lists have not yet been passed through the upstream MI-Pack annotation searches
  (Empirical Formulae Search, Single-Peak Search/Transformation Mapping, Peak-Pattern
  Search)
- You need univariate or multivariate statistics rather than metabolite annotation
  consolidation — the Galaxy-M workflow explicitly does not include such statistics
  at this stage
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MI-Pack
  role: Provides the Combine Outputs tool that consolidates single or combinations
    of SQLite annotation search outputs into a summary file
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Wraps MI-Pack tools including Combine Outputs as Galaxy tool wrappers so the
    consolidation step runs as part of the annotation workflow
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform hosting the Galaxy-M annotation workflow in which Combine Outputs
    executes
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:bc61db08014104b4bec1cc11764cca69084cc4895c55dad6cdc358e1bb14d936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# SQLite annotation results consolidation

## Summary

Consolidates the SQLite outputs of MI-Pack's annotation searches (Empirical Formulae Search, Single-Peak Search/Transformation Mapping, Peak-Pattern Search) into a single summary annotation database using the MI-Pack Combine Outputs tool within the Galaxy-M workflow.

## When to use

Use when multiple MI-Pack annotation searches have written SQLite output files and a consolidated summary file of metabolite annotations is needed, or when a single SQLite output needs to be summarized directly.

## When NOT to use

- Peak lists have not yet been passed through the upstream MI-Pack annotation searches (Empirical Formulae Search, Single-Peak Search/Transformation Mapping, Peak-Pattern Search)
- You need univariate or multivariate statistics rather than metabolite annotation consolidation — the Galaxy-M workflow explicitly does not include such statistics at this stage

## Inputs

- MI-Pack SQLite output files from Empirical Formulae Search
- MI-Pack SQLite output files from Single-Peak Search and Transformation Mapping
- MI-Pack SQLite output files from Peak-Pattern Search

## Outputs

- Consolidated summary annotation file (SQLite), e.g. mi_pack_annotations.sqlite

## How to apply

After running MI-Pack's Empirical Formulae Search, Single-Peak Search/Transformation Mapping and Peak-Pattern Search — each of which writes its results as SQLite files — invoke the MI-Pack Combine Outputs tool to merge these into a final summary file. Combine Outputs accepts either a single SQLite output or combinations of them, so you decide per dataset whether to consolidate all search types or a subset. In the Galaxy-M case studies the consolidated result is saved as mi_pack_annotations.sqlite. Because Single-Peak Search and Transformation Mapping query compound databases (KEGG, HMDB and LIPID MAPS), verify the consolidated annotations reflect the intended database coverage; optionally enable Parallel Python 1.6.4 beforehand so upstream Empirical Formula search can run in parallel on multi-core systems. Correctness is judged by the summary file containing the union of annotations from the constituent search outputs and being loadable as a valid SQLite database.

## Related tools

- **MI-Pack** (Provides the Combine Outputs tool that consolidates single or combinations of SQLite annotation search outputs into a summary file) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Wraps MI-Pack tools including Combine Outputs as Galaxy tool wrappers so the consolidation step runs as part of the annotation workflow) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform hosting the Galaxy-M annotation workflow in which Combine Outputs executes) — https://github.com/galaxyproject/galaxy

## Examples

```
python -m mipack.combine_outputs --sqlite ef_search.sqlite sp_search.sqlite pp_search.sqlite --output mi_pack_annotations.sqlite
```

## Evaluation signals

- The consolidated output is a valid SQLite database that opens without error
- The summary file contains annotations traceable to each constituent MI-Pack search output (formulae, single-peak/transformation, peak-pattern)
- Consolidated annotations reference the expected compound databases (KEGG, HMDB, LIPID MAPS)
- The output filename/content matches the published Galaxy-M case-study convention (mi_pack_annotations.sqlite)

## Limitations

- Consolidation is only as complete as the upstream searches; peaks failing Empirical Formulae Search's mass-error tolerance will be absent
- Case-study DIMS transient (.dat) files were not in the MetaboLights accession, so reproducing upstream searches for MTBLS79 requires obtaining them from the original authors
- The upstream Empirical Formula search benefits from Parallel Python 1.6.4 on multi-core systems; without it, searches are slower
- License is GPLv3 for MI-Pack and Galaxy-M tool files

## Evidence

- [other] Combine Outputs produces a summary file from single or combinations of SQLite outputs: [evidence span withheld — anchor status: not_found]
- [other] Save consolidated annotations as mi_pack_annotations.sqlite: [evidence span withheld — anchor status: not_found]
- [abstract] Searches are against KEGG, HMDB and LIPID MAPS: [evidence span withheld — anchor status: not_found]
- [readme] MI-Pack version pinning in Galaxy-M README: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
- [readme] Parallel Python speeds up Empirical Formula search: [evidence span withheld — anchor status: external_doc]
