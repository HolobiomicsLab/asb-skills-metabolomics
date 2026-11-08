---
name: dims-lcms-peak-list-annotation
description: Use when a peak list that has already been peak-picked, aligned, and
  filter-cleaned (DIMS SIM-stitched or XCMS-processed) needs metabolite annotation
  by accurate mass, adduct and isotopic-abundance patterns, and compound-database
  lookup.
when_to_use_negative:
- Input is still raw instrument data (e.g. DIMS transient .dat files or LC-MS vendor
  files) that has not been peak-picked, aligned and filtered — run the Galaxy-M processing
  stages first
- The goal is statistical analysis (PCA, separation testing) rather than metabolite
  annotation — use the Galaxy-M PLS-Toolbox/PCA stage instead
- Univariate statistics are required — the article notes no univariate statistics
  tools are included in the current implementation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MI-Pack
  role: 'Provides the annotation tools themselves: Empirical Formulae Search, Single-Peak
    Search and Transformation Mapping, Peak-Pattern Search, and Combine Outputs'
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Supplies the Galaxy tool wrappers (.xml) and config files that expose the
    MI-Pack annotation tools as workflow steps
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Workflow platform in which the MI-Pack annotation tools are chained and run
    with provenance/history tracking
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:82cd805223a2b5dc315588d91aa4d3117436ab6b492af480ea25a5bcc2f869f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# dims-lcms-peak-list-annotation

## Summary

Annotates metabolites on processed DIMS/LC-MS peak lists using the MI-Pack toolset inside Galaxy-M: empirical formula assignment from accurate masses, database matching against KEGG/HMDB/LIPID MAPS, and adduct/isotopic-pattern annotation consolidated into SQLite outputs.

## When to use

Use when a peak list that has already been peak-picked, aligned, and filter-cleaned (DIMS SIM-stitched or XCMS-processed) needs metabolite annotation by accurate mass, adduct and isotopic-abundance patterns, and compound-database lookup.

## When NOT to use

- Input is still raw instrument data (e.g. DIMS transient .dat files or LC-MS vendor files) that has not been peak-picked, aligned and filtered — run the Galaxy-M processing stages first
- The goal is statistical analysis (PCA, separation testing) rather than metabolite annotation — use the Galaxy-M PLS-Toolbox/PCA stage instead
- Univariate statistics are required — the article notes no univariate statistics tools are included in the current implementation

## Inputs

- Cleaned DIMS peak list (post Sum Transients, SimStitch, Align Samples, replicate/blank/sample filtering)
- Cleaned LC-MS peak/feature list from XCMS-Basic-Birmingham-Pipe with replicate, blank and sample filtering
- Compound reference databases: KEGG, HMDB, LIPID MAPS
- Configured mass-error tolerance for formula search

## Outputs

- SQLite outputs from each MI-Pack search (Empirical Formulae, Single-Peak Search/Transformation Mapping, Peak-Pattern Search)
- Combined summary annotation database (e.g. mi_pack_annotations.sqlite) from Combine Outputs

## How to apply

Feed the cleaned peak list into MI-Pack's Empirical Formulae Search, which matches accurate masses to elemental compositions within a configured mass-error tolerance (optionally in parallel via Parallel Python 1.6.4). Run Single-Peak Search and Transformation Mapping against the KEGG, HMDB and LIPID MAPS compound reference databases to assign putative metabolite identities and biochemical transformations. Run Peak-Pattern Search to extract adduct patterns and relative isotopic abundances across co-occurring peaks, giving additional evidence beyond accurate mass alone. Finally run Combine Outputs to consolidate the individual SQLite search outputs, single or in combination, into a summary annotation database (e.g. mi_pack_annotations.sqlite). In Galaxy-M, each of these is exposed as a wrapped tool; cross-check the run against the published Galaxy-M histories/workflows to confirm reproducibility of the case-study output.

## Related tools

- **MI-Pack** (Provides the annotation tools themselves: Empirical Formulae Search, Single-Peak Search and Transformation Mapping, Peak-Pattern Search, and Combine Outputs) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Supplies the Galaxy tool wrappers (.xml) and config files that expose the MI-Pack annotation tools as workflow steps) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Workflow platform in which the MI-Pack annotation tools are chained and run with provenance/history tracking) — https://github.com/galaxyproject/galaxy

## Examples

```
git clone https://github.com/Viant-Metabolomics/MI-Pack && cd MI-Pack && python setup.py build && python setup.py install
```

## Evaluation signals

- Each MI-Pack search tool emits a SQLite output; verify all expected SQLite outputs (formula, database match, peak-pattern) exist before running Combine Outputs
- Combine Outputs must consolidate the individual SQLite outputs into a single summary annotation file without dropping records from the constituent searches
- Cross-check the resulting annotations against the published Galaxy-M histories/workflows from the GigaScience case study (MTBLS79 DIMS, MTBLS146 LC-MS subsets) to confirm the run reproduces the case-study output
- Adduct and relative isotopic abundance annotations from Peak-Pattern Search should be consistent with the co-occurring peaks in the input peak list
- Pin the MI-Pack version used: the GigaScience publication utilises MI-Pack GitHub commit 06ce0ace643ee1cf1d27550769a5272b7ea50825

## Limitations

- MI-Pack runs under Python 2.7; Parallel Python 1.6.4 is non-essential but needed to parallelise Empirical Formula search on multi-core systems
- The specific MI-Pack version tested is a pinned GitHub commit (06ce0ace643ee1cf1d27550769a5272b7ea50825) tested against a pinned Galaxy commit (c429777c93680dcee449fe410f5360afbe673758); newer versions may not be compatible
- Annotation confidence rests on accurate mass plus adduct/isotopic evidence against KEGG/HMDB/LIPID MAPS; it is putative identification, not validated structural confirmation

## Evidence

- [abstract] Metabolite annotation is performed using MI-Pack [10].: "Metabolite annotation is performed using MI-Pack [10]."
- [abstract] compound reference databases (e.g. KEGG, HMDB and LIPID MAPS): "compound reference databases (e.g. KEGG, HMDB and LIPID MAPS)"
- [methods] parallel processing to speed up e.g. Empirical Formula search: "parallel processing to speed up e.g. Empirical Formula search"
- [methods] Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack: "Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack"
- [readme] The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
- [readme] Non-essential: Parallel python (version 1.6.4, [http://www.parallelpython.com/]): [evidence span withheld — anchor status: external_doc]
