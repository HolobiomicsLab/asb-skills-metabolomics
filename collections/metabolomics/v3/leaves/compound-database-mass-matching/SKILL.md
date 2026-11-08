---
name: compound-database-mass-matching
description: Use when peak lists (DIMS or LC-MS) already contain accurate m/z values
  and you must assign putative metabolite identities via empirical formulae, single-peak
  matches, transformation mapping, or adduct/isotopic peak-pattern evidence.
when_to_use_negative:
- Input is raw spectra or an already-annotated metabolite table — this skill operates
  on processed peak lists requiring putative identities, not raw transients or curated
  annotations.
- The goal is statistical analysis (PCA, normalization) or raw peak picking/alignment
  (XCMS, SimStitch) rather than metabolite annotation.
- Only one search output is needed and combining multiple SQLite results would obscure,
  rather than summarize, the annotation.
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MI-Pack
  role: Performs Empirical Formulae Search, Single-Peak Search, Transformation Mapping,
    Peak-Pattern Search and Combine Outputs to annotate peaks against compound databases
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Galaxy wrapper layer exposing the MI-Pack annotation tools as reproducible
    workflow steps
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: Galaxy
  role: Platform in which the MI-Pack annotation tools are chained and their outputs
    cross-checked against published histories/workflows
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:dd19e8b6c7324d251920204b6b12ba1859219a5b1657536f80dc0c1e8b8a2f5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy
  - Galaxy-M
  - MI-Pack
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# compound-database-mass-matching

## Summary

Annotating metabolite peaks by matching accurate masses and elemental formulae against compound reference databases (KEGG, HMDB, LIPID MAPS) using the MI-Pack toolset inside the Galaxy-M workflow.

## When to use

Use when peak lists (DIMS or LC-MS) already contain accurate m/z values and you must assign putative metabolite identities via empirical formulae, single-peak matches, transformation mapping, or adduct/isotopic peak-pattern evidence.

## When NOT to use

- Input is raw spectra or an already-annotated metabolite table — this skill operates on processed peak lists requiring putative identities, not raw transients or curated annotations.
- The goal is statistical analysis (PCA, normalization) or raw peak picking/alignment (XCMS, SimStitch) rather than metabolite annotation.
- Only one search output is needed and combining multiple SQLite results would obscure, rather than summarize, the annotation.

## Inputs

- Processed peak list(s) with accurate m/z values from DIMS or LC-MS data (e.g. Galaxy-M case-study peak lists derived from MetaboLights MTBLS79 and MTBLS146)
- Compound reference databases: KEGG, HMDB, LIPID MAPS
- MI-Pack python package (github commit 06ce0ace643ee1cf1d27550769a5272b7ea50825)

## Outputs

- SQLite annotation outputs from each MI-Pack search (formula, single-peak, transformation, peak-pattern)
- Consolidated summary annotation file, e.g. mi_pack_annotations.sqlite

## How to apply

First run MI-Pack Empirical Formulae Search on each peak list to assign elemental compositions within a configured mass-error tolerance (optionally installing Parallel Python 1.6.4 to parallelize this search across cores). Then run MI-Pack Single-Peak Search and Transformation Mapping against compound databases such as KEGG, HMDB and LIPID MAPS to link formulae to putative metabolites and biotransformations. Run Peak-Pattern Search to extract adduct patterns and relative isotopic abundances across co-occurring peaks, which raises confidence beyond single-peak matching. Finally run MI-Pack Combine Outputs to consolidate the individual SQLite search outputs, whether from a single search or combinations of them, into a summary annotation file (e.g. mi_pack_annotations.sqlite) for cross-checking against published Galaxy-M histories.

## Related tools

- **MI-Pack** (Performs Empirical Formulae Search, Single-Peak Search, Transformation Mapping, Peak-Pattern Search and Combine Outputs to annotate peaks against compound databases) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Galaxy wrapper layer exposing the MI-Pack annotation tools as reproducible workflow steps) — https://github.com/Viant-Metabolomics/Galaxy-M
- **Galaxy** (Platform in which the MI-Pack annotation tools are chained and their outputs cross-checked against published histories/workflows) — https://github.com/galaxyproject/galaxy

## Examples

```
python setup.py build && python setup.py install
```

## Evaluation signals

- Each peak list yields an Empirical Formulae Search output whose assigned compositions fall within the configured mass-error tolerance.
- Single-Peak Search and Transformation Mapping each produce SQLite outputs referencing the loaded KEGG, HMDB and LIPID MAPS databases.
- Peak-Pattern Search reports adduct patterns and relative isotopic abundances consistent with co-occurring peaks in the input peak list.
- Combine Outputs successfully consolidates single or combined SQLite inputs into one summary file (mi_pack_annotations.sqlite) without missing search results.
- Resulting annotations match the published Galaxy-M case-study histories/workflows when re-run on the MTBLS79/MTBLS146-derived test data.

## Limitations

- Parallel Python 1.6.4 is only a non-essential optional dependency; without it the Empirical Formula search runs serially on multi-core systems.
- The annotation stage depends on the specific MI-Pack GitHub commit (06ce0ace643ee1cf1d27550769a5272b7ea50825) used by the GigaScience publication; other versions may not reproduce results.
- Annotation confidence rests on matching against the selected compound databases (KEGG, HMDB, LIPID MAPS); metabolites absent from those databases cannot be annotated.

## Evidence

- [other] Single-Peak Search and Transformation Mapping search these against compound databases (e.g. KEGG, HMDB, LIPID MAPS): [evidence span withheld — anchor status: not_found]
- [other] Empirical Formulae Search matches accurate masses to elemental compositions: [evidence span withheld — anchor status: not_found]
- [other] Peak-Pattern Search extracts adduct patterns: [evidence span withheld — anchor status: not_found]
- [other] Combine Outputs produces a summary file: [evidence span withheld — anchor status: not_found]
- [readme] MI-Pack version used by the publication: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
- [readme] Parallel python optional dependency: [evidence span withheld — anchor status: external_doc]
