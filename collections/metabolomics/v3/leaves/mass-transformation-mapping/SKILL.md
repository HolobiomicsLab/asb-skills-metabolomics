---
name: mass-transformation-mapping
description: Use when peak lists (e.g. DIMS MTBLS79 or LC-MS MTBLS146 subsets) have
  already been formula-assigned by Empirical Formulae Search and you must link those
  accurate-mass formulae to putative metabolites in compound reference databases.
when_to_use_negative:
- When the peak list has not yet been processed by Empirical Formulae Search — Transformation
  Mapping searches empirically assigned formulae, not raw masses
- When no compound reference database (KEGG, HMDB, LIPID MAPS) is installed, since
  the search depends on these resources
- When the goal is raw-data processing (peak picking, alignment, filtering) rather
  than metabolite annotation
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MI-Pack
  role: Provides the Transformation Mapping tool used to search empirically assigned
    formulae against compound databases
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Wraps the MI-Pack annotation tools as Galaxy tools/workflow steps
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:9da9f6ba147ecee2e30a87cde2d2e191a2ef530418364ae242c356fcdcf30784
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy-M
  - MI-Pack
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-transformation-mapping

## Summary

Applies MI-Pack Transformation Mapping within the Galaxy-M annotation stage to search empirically assigned elemental formulae against compound reference databases (e.g. KEGG, HMDB, LIPID MAPS), producing an SQLite annotation database that is consolidated via Combine Outputs.

## When to use

Use when peak lists (e.g. DIMS MTBLS79 or LC-MS MTBLS146 subsets) have already been formula-assigned by Empirical Formulae Search and you must link those accurate-mass formulae to putative metabolites in compound reference databases.

## When NOT to use

- When the peak list has not yet been processed by Empirical Formulae Search — Transformation Mapping searches empirically assigned formulae, not raw masses
- When no compound reference database (KEGG, HMDB, LIPID MAPS) is installed, since the search depends on these resources
- When the goal is raw-data processing (peak picking, alignment, filtering) rather than metabolite annotation

## Inputs

- Peak list with empirical formulae assigned by MI-Pack Empirical Formulae Search (from DIMS SIM-stitched or XCMS-processed LC-MS data)
- Compound reference database files: KEGG, HMDB, LIPID MAPS
- Output from MI-Pack Single-Peak Search and/or Peak-Pattern Search (SQLite outputs) for combination

## Outputs

- SQLite annotation database mapping peaks to putative metabolites
- Consolidated annotation summary file from MI-Pack Combine Outputs (e.g. mi_pack_annotations.sqlite)

## How to apply

Within the Galaxy-M annotation workflow, first run MI-Pack Empirical Formulae Search on the aligned peak list to assign elemental compositions within the configured mass-error tolerance, optionally enabling Parallel Python 1.6.4 for multi-core speed-up. Then run Transformation Mapping (alongside Single-Peak Search) against the KEGG, HMDB and LIPID MAPS compound databases so each formula-matched peak is associated with database compounds; Transformation Mapping searches the empirically assigned formulae rather than raw m/z values. Peak-Pattern Search can add adduct and isotopic-abundance evidence across co-occurring peaks. Finally, run MI-Pack Combine Outputs to consolidate the single or combined SQLite search outputs into one summary file (saved e.g. as mi_pack_annotations.sqlite). Success is judged by the completeness of the annotation database and by cross-checking the run against the published Galaxy-M histories/workflows from the GigaScience paper (commit 06ce0ace643ee1cf1d27550769a5272b7ea50825 of MI-Pack).

## Related tools

- **MI-Pack** (Provides the Transformation Mapping tool used to search empirically assigned formulae against compound databases) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Wraps the MI-Pack annotation tools as Galaxy tools/workflow steps) — https://github.com/Viant-Metabolomics/Galaxy-M

## Evaluation signals

- The output SQLite database contains entries linking peak formulae to KEGG/HMDB/LIPID MAPS compound identifiers
- Running Combine Outputs over the single or combined SQLite outputs produces a summary file without orphaned search outputs
- Cross-checking the history against the published Galaxy-M workflows/histories reproduces the case-study annotation output
- MI-Pack is installed at the pinned commit 06ce0ace643ee1cf1d27550769a5272b7ea50825 to match the published run

## Limitations

- The search is limited to compounds present in the selected reference databases (KEGG, HMDB, LIPID MAPS)
- Parallel Python (version 1.6.4) is non-essential but speeds up the Empirical Formula search; without it, multi-core parallelisation is not available
- The published Galaxy-M workflow relies on a specific MI-Pack commit; other versions may not reproduce the paper's annotations

## Evidence

- [abstract] Transformation Mapping searches these against compound databases: [evidence span withheld — anchor status: not_found]
- [abstract] Metabolite annotation performed using MI-Pack: "Metabolite annotation is performed using MI-Pack [10]."
- [abstract] Combine Outputs consolidates SQLite search outputs: [evidence span withheld — anchor status: not_found]
- [methods] Parallel Python speeds up Empirical Formula search: "parallel processing to speed up e.g. Empirical Formula search"
- [methods] Download MI-Pack python package: "Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack"
- [readme] MI-Pack pinned commit for reproducibility: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
- [abstract] Reference databases used for annotation: "compound reference databases (e.g. KEGG, HMDB and LIPID MAPS)"
