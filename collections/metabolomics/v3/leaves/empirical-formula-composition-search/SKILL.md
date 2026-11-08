---
name: empirical-formula-composition-search
description: Use when a processed, aligned peak list (DIMS or LC-MS) with accurate-mass
  m/z values needs elemental composition assignments before compound-database searches
  like Single-Peak Search or Peak-Pattern Search.
when_to_use_negative:
- Input is still raw instrument data (Thermo .dat transients or vendor files) rather
  than a processed, calibrated, aligned peak list — run Sum Transients/Process Transients/Mass
  Calibration or XCMS first
- The goal is compound-level identification already supported by existing formula
  assignments — use MI-Pack Single-Peak Search or Transformation Mapping instead
- Peak m/z values come from low-resolution data without sufficient mass accuracy to
  constrain elemental compositions within the tolerance
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MI-Pack
  role: Provides the Empirical Formulae Search tool that matches accurate masses to
    elemental compositions within an error tolerance
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Wraps the Empirical Formula search as a Galaxy tool within the metabolomics
    workflow, with .xml wrapper and config files
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the Empirical Formula search step is executed and reproducibly
    recorded in histories
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:d90c87bf733d92e8e704f72b6a7745272682c82476e975d15f6b617a73399e9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# empirical-formula-composition-search

## Summary

Assign elemental compositions to peak-list m/z values by matching accurate masses to candidate empirical formulae within a user-defined mass-error tolerance, as the first step of MI-Pack metabolite annotation in the Galaxy-M workflow for DIMS and LC-MS metabolomics.

## When to use

Use when a processed, aligned peak list (DIMS or LC-MS) with accurate-mass m/z values needs elemental composition assignments before compound-database searches like Single-Peak Search or Peak-Pattern Search.

## When NOT to use

- Input is still raw instrument data (Thermo .dat transients or vendor files) rather than a processed, calibrated, aligned peak list — run Sum Transients/Process Transients/Mass Calibration or XCMS first
- The goal is compound-level identification already supported by existing formula assignments — use MI-Pack Single-Peak Search or Transformation Mapping instead
- Peak m/z values come from low-resolution data without sufficient mass accuracy to constrain elemental compositions within the tolerance

## Inputs

- Galaxy-M peak list (processed peak table from DIMS SIM-stitching or XCMS-Basic-Birmingham-Pipe alignment) with accurate-mass m/z values
- User-configured mass-error tolerance
- Optional: Parallel Python 1.6.4 environment for multi-core execution

## Outputs

- SQLite output of empirical formula assignments per peak (input to MI-Pack Single-Peak Search, Transformation Mapping and Combine Outputs)

## How to apply

Run the MI-Pack Empirical Formulae Search tool (installed from Viant-Metabolomics/MI-Pack, optionally with Parallel Python 1.6.4 for multi-core execution) on each Galaxy-M peak list. The tool matches each accurate mass to elemental compositions within a configured mass-error tolerance, exploiting the high mass accuracy of the DIMS SIM-stitching and XCMS-processed LC-MS peak tables. Results are written as SQLite outputs that feed downstream MI-Pack annotation steps — Single-Peak Search and Transformation Mapping against KEGG, HMDB and LIPID MAPS, and Peak-Pattern Search for adduct/isotopic patterns. Formula assignments should be sanity-checked against the published Galaxy-M case-study histories (MTBLS79 DIMS and MTBLS146 LC-MS subsets) before consolidation with Combine Outputs into mi_pack_annotations.sqlite.

## Related tools

- **MI-Pack** (Provides the Empirical Formulae Search tool that matches accurate masses to elemental compositions within an error tolerance) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Wraps the Empirical Formula search as a Galaxy tool within the metabolomics workflow, with .xml wrapper and config files) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the Empirical Formula search step is executed and reproducibly recorded in histories) — https://github.com/galaxyproject/galaxy

## Examples

```
python setup.py build; python setup.py install  # install MI-Pack (optionally with Parallel Python 1.6.4) then run the Empirical Formulae Search Galaxy tool on each peak list
```

## Evaluation signals

- Each peak in the input list receives either a formula assignment or an explicit no-match within the configured mass-error tolerance (no silent drops)
- Assignments reproduce the published Galaxy-M case-study results when cross-checked against the published histories/workflows for MTBLS79 and MTBLS146
- Output SQLite file is readable by downstream MI-Pack tools (Single-Peak Search, Peak-Pattern Search, Combine Outputs)
- Consolidated Combine Outputs file (mi_pack_annotations.sqlite) contains formula-annotated peaks consistent with the input peak list
- Pinned software provenance: MI-Pack commit 06ce0ace643ee1cf1d27550769a5272b7ea50825, Galaxy commit c429777c93680dcee449fe410f5360afbe673758

## Limitations

- Parallel Python 1.6.4 is non-essential and only needed to parallelize the Empirical Formula search on multi-core systems; without it the step runs single-threaded
- Search is constrained by compound reference databases (KEGG, HMDB, LIPID MAPS) and the configured mass-error tolerance; formulae outside these are not reported
- Requires sufficient accurate mass; low-resolution or poorly calibrated peak lists will yield ambiguous or incorrect compositions
- Python 2.7 / MATLAB 2014a era software stack — modern Python 3 environments may be incompatible without the pre-installed GigaDB VM

## Evidence

- [results] Empirical Formulae Search matches accurate masses to elemental compositions within an error tolerance: [evidence span withheld — anchor status: not_found]
- [methods] parallel processing speeds the formula search: "parallel processing to speed up e.g. Empirical Formula search"
- [readme] Parallel Python is optional for MI-Pack: "Non-essential

* Parallel python (version 1.6.4, [http://www.parallelpython.com/])"
- [abstract] metabolite annotation is performed using MI-Pack: "Metabolite annotation is performed using MI-Pack [10]."
- [readme] pinned MI-Pack commit used by the publication: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
