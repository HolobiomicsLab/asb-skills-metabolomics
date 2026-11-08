---
name: adduct-isotope-pattern-annotation
description: Use when a processed peak list (post peak-picking/alignment, e.g. from
  XCMS or the DIMS SIM-stitch pipeline) needs metabolite annotations where adduct
  forms and isotopic abundance patterns across co-eluting/co-occurring peaks should
  be exploited rather than matching single masses alone.
when_to_use_negative:
- Input is raw mass-spectrometry data (e.g. DIMS .dat transient files or LC-MS vendor
  files) rather than a processed peak list — peak picking, alignment and filtering
  must happen first.
- The goal is statistical analysis (normalization, scaling, PCA) — Galaxy-M's downstream
  statistics tools, not MI-Pack annotation, apply.
- Only a single-peak accurate-mass match is required with no interest in adduct/isotopic
  pattern corroboration — Single-Peak Search alone suffices.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MI-Pack
  role: Provides Peak-Pattern Search (plus Empirical Formulae Search, Single-Peak
    Search/Transformation Mapping, Combine Outputs) that performs the adduct and isotopic-abundance
    annotation on Galaxy-M peak lists
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Galaxy tool wrappers and workflows that expose the MI-Pack annotation tools
    as steps of the end-to-end DIMS/LC-MS metabolomics workflow
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the MI-Pack annotation tools are wrapped and executed as
    reproducible workflow steps
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:c2e2aebde2154fdeeae2bc3399ca4dc4bee4ada0145549e2f1f7c8b6ea912e28
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# adduct-isotope-pattern-annotation

## Summary

Annotates metabolite features in DIMS/LC-MS peak lists by extracting adduct patterns and relative isotopic abundances across co-occurring peaks using MI-Pack's Peak-Pattern Search within the Galaxy-M workflow, boosting confidence in metabolite identifications beyond accurate mass alone.

## When to use

Use when a processed peak list (post peak-picking/alignment, e.g. from XCMS or the DIMS SIM-stitch pipeline) needs metabolite annotations where adduct forms and isotopic abundance patterns across co-eluting/co-occurring peaks should be exploited rather than matching single masses alone.

## When NOT to use

- Input is raw mass-spectrometry data (e.g. DIMS .dat transient files or LC-MS vendor files) rather than a processed peak list — peak picking, alignment and filtering must happen first.
- The goal is statistical analysis (normalization, scaling, PCA) — Galaxy-M's downstream statistics tools, not MI-Pack annotation, apply.
- Only a single-peak accurate-mass match is required with no interest in adduct/isotopic pattern corroboration — Single-Peak Search alone suffices.

## Inputs

- Processed peak list from the Galaxy-M DIMS or LC-MS pipeline (post peak picking, alignment, and replicate/blank/sample filtering)
- Elemental composition assignments from MI-Pack Empirical Formulae Search
- Compound reference databases: KEGG, HMDB, LIPID MAPS

## Outputs

- SQLite annotation database from Peak-Pattern Search containing adduct patterns and relative isotopic abundances
- Consolidated annotation summary file from Combine Outputs (e.g. mi_pack_annotations.sqlite)

## How to apply

First run MI-Pack's Empirical Formulae Search on each cleaned peak list to assign elemental compositions within the configured mass-error tolerance. Then run Single-Peak Search and Transformation Mapping against compound reference databases (KEGG, HMDB, LIPID MAPS) before running Peak-Pattern Search, which extracts adduct patterns and relative isotopic abundances across groups of co-occurring peaks. Peak-Pattern Search writes its results as a SQLite database; consolidate these SQLite outputs (single or combined) with MI-Pack's Combine Outputs tool to produce the final annotation summary file (e.g. mi_pack_annotations.sqlite). Optionally install Parallel Python 1.6.4 so the Empirical Formula search can run in parallel on multi-core systems. Optionally install Parallel Python 1.6.4 so the Empirical Formula search can run in parallel on multi-core systems. Correctness is judged by cross-checking the resulting annotations against the published Galaxy-M histories/workflows for the MTBLS79 (DIMS) and MTBLS146 (LC-MS) case studies.

## Related tools

- **MI-Pack** (Provides Peak-Pattern Search (plus Empirical Formulae Search, Single-Peak Search/Transformation Mapping, Combine Outputs) that performs the adduct and isotopic-abundance annotation on Galaxy-M peak lists) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Galaxy tool wrappers and workflows that expose the MI-Pack annotation tools as steps of the end-to-end DIMS/LC-MS metabolomics workflow) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the MI-Pack annotation tools are wrapped and executed as reproducible workflow steps) — https://github.com/galaxyproject/galaxy

## Examples

```
python setup.py install  # install MI-Pack; then in Galaxy-M run Empirical Formulae Search -> Single-Peak Search/Transformation Mapping (KEGG, HMDB, LIPID MAPS) -> Peak-Pattern Search -> Combine Outputs to produce mi_pack_annotations.sqlite
```

## Evaluation signals

- Peak-Pattern Search produces a SQLite database of adduct patterns and relative isotopic abundances for co-occurring peaks — verify the SQLite output is non-empty and well-formed.
- Combine Outputs must consolidate single or combinations of SQLite outputs into a final summary file (mi_pack_annotations.sqlite) without missing per-tool tables.
- Cross-check annotations against the published Galaxy-M histories/workflows for the MTBLS79 (DIMS) and MTBLS146 (LC-MS) case studies to confirm the run reproduces the case-study output.
- Empirical Formula assignments upstream must fall within the configured mass-error tolerance before pattern annotation is trusted.
- MI-Pack version consistency: the publication uses MI-Pack Github commit 06ce0ace643ee1cf1d27550769a5272b7ea50825; deviations may change annotation output.

## Limitations

- Pattern annotation is only as good as upstream peak detection: peaks must survive replicate filtering (x-out-of-n technical replicates), blank filtering, and sample filtering before annotation.
- The DIMS transient (.dat) source files were not included in the MetaboLights accession and had to be obtained from the original authors, complicating end-to-end reproduction of the annotation inputs.
- SimStitch/MCR-related tooling requires WINE in 32-bit mode (WINEARCH=win32); otherwise tools fail with errors like 'Invalid file identifier' when opening DIMS data.
- Compound annotation depends on the completeness of KEGG, HMDB and LIPID MAPS reference databases; metabolites outside these are unannotated.

## Evidence

- [methods] Peak-Pattern Search extracts adduct patterns and relative isotopic abundances: [evidence span withheld — anchor status: not_found]
- [methods] Single-Peak Search and Transformation Mapping search against compound databases (e.g. KEGG, HMDB, LIPID MAPS): [evidence span withheld — anchor status: not_found]
- [methods] Combine Outputs produces a summary file from single or combinations of SQLite outputs: [evidence span withheld — anchor status: not_found]
- [abstract] Metabolite annotation is performed using MI-Pack [10].: "Metabolite annotation is performed using MI-Pack [10]."
- [methods] Empirical Formulae Search matches accurate masses to elemental compositions within an error tolerance: [evidence span withheld — anchor status: not_found]
- [readme] The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
