---
name: annotation-confidence-level-assignment
description: Use when empCpd JSON from pcpfm has MS2 spectra already mapped to features/empirical
  compounds and reference .msp spectral libraries are available, and Level 2 identifications
  are needed.
when_to_use_negative:
- Input is a raw feature table without MS2 spectra mapped to empirical compounds —
  run the map_ms2 step first or use MS1-only annotation
- The goal is authentic-standard confirmation (Level 1) annotation, which requires
  a standard library rather than a public MS2 reference database
- Working outside the pcpfm/empCpd data model where the JSON empCpd list and its moniker
  chain are not the annotation container
edam_operation: http://edamontology.org/operation_3803
tools:
- name: pcpfm
  role: Pipeline providing the l2_annotate operation that searches mapped MS2 spectra
    against reference libraries and writes annotated empCpd JSON
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: MatchMS
  role: Computes spectral similarity between mapped experimental MS2 spectra and library
    entries, with interval tree optimization
- name: MoNA
  role: Default reference MS2 spectral database (Orbitrap files) downloaded via 'pcpfm
    download_extras'
- name: MetDataModel
  role: Defines the empirical compound (empCpd) data structure that carries the annotation
    levels
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:93a37edcdfad1478e2c73aeba42bf94dbef7d2c94ca75cad11de31c75aa12bc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - MatchMS
  - MetDataModel
  - MoNA
  - Python
  - pcpfm
  edam_operation: http://edamontology.org/operation_3803
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# annotation-confidence-level-assignment

## Summary

Assigns metabolite annotation confidence levels to empirical compounds (empCpds) in the pcpfm pipeline, attaching Level 2 (reference MS2 spectral similarity) identifications by matching mapped experimental MS2 spectra against a reference library such as MoNA via matchms, complementing MS1 and authentic-standard annotations.

## When to use

Use when empCpd JSON from pcpfm has MS2 spectra already mapped to features/empirical compounds and reference .msp spectral libraries are available, and Level 2 identifications are needed.

## When NOT to use

- Input is a raw feature table without MS2 spectra mapped to empirical compounds — run the map_ms2 step first or use MS1-only annotation
- The goal is authentic-standard confirmation (Level 1) annotation, which requires a standard library rather than a public MS2 reference database
- Working outside the pcpfm/empCpd data model where the JSON empCpd list and its moniker chain are not the annotation container

## Inputs

- MS2-mapped empirical compound JSON (e.g. moniker 'MS2_mapped') from the pcpfm map_ms2 step
- Reference tandem MS spectral library .msp files (MoNA Orbitrap databases or user-supplied) for positive mode
- Reference tandem MS spectral library .msp files for negative mode

## Outputs

- Updated empCpd JSON under a new moniker (e.g. 'MS2_MS1_annotated') carrying Level 2 MS2 annotations
- Annotation table mapping empirical compounds to best-matching reference library entries

## How to apply

Load the MS2-mapped empCpd JSON produced by the map_ms2 step, plus reference tandem MS libraries as .msp files (default: MoNA Orbitrap databases fetched via 'pcpfm download_extras'), selecting separate files for positive and negative ionization modes via --msp_files_pos / --msp_files_neg. Use matchms to compute spectral similarity between each mapped experimental MS2 spectrum and library entries, filtering spectra by --ms2_min_peaks (minimum peak count required for a spectrum to be searched and matched) and honoring any --ms2_similarity_metric override. Attach the best matching library entries to the empirical compounds as Level 2 (similarity to reference MS2 spectra) annotations and write the updated empCpd list under a new moniker (e.g. 'MS2_MS1_annotated') as JSON. The empCpd structure is chainable, so Level 2 assignments layer onto existing MS1/pre-annotation states and can later be upgraded to authentic-standard (Level 1) annotations. Rationale: annotation assigns compound identification to the empirical compound, a computational unit that accommodates different levels or probable scores.

## Related tools

- **pcpfm** (Pipeline providing the l2_annotate operation that searches mapped MS2 spectra against reference libraries and writes annotated empCpd JSON) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **MatchMS** (Computes spectral similarity between mapped experimental MS2 spectra and library entries, with interval tree optimization)
- **MoNA** (Default reference MS2 spectral database (Orbitrap files) downloaded via 'pcpfm download_extras')
- **MetDataModel** (Defines the empirical compound (empCpd) data structure that carries the annotation levels) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm l2_annotate --empCpd_file annotations/empCpd.json --msp_files_pos MoNA_pos.msp --msp_files_neg MoNA_neg.msp --ms2_min_peaks 5
```

## Evaluation signals

- The output empCpd JSON exists and parses successfully
- Each reported Level 2 annotation corresponds to a reference MS2 spectrum match that met the minimum-peak-count requirement (--ms2_min_peaks)
- Each match satisfied the configured similarity criterion (--ms2_similarity_metric)
- Annotations attach to the correct ionization mode: matches came from --msp_files_pos or --msp_files_neg consistent with the empCpd's mode
- The empCpd JSON remains chainable: prior MS1/pre-annotation assignments are preserved alongside the new Level 2 entries

## Limitations

- Only spectra meeting the minimum peak count are searched and matched; low-quality MS2 spectra yield no Level 2 annotation
- Empirical compounds may represent unresolved isomers, so Level 2 spectral matches cannot distinguish isomeric structures
- HMDB and other annotation sources are not redistributed and require license agreement before downloading via 'pcpfm download_extras'

## Evidence

- [readme] Level 2 MS2 annotation via matchms against MoNA and empCpd levels: "perform MS2 annotation (matchms) using a custom database (default MoNA)"
- [readme] empCpd annotation levels rationale: [evidence span withheld — anchor status: external_doc]
- [intro] Annotation sources: "Annotation is a major step after preprocessing, utilizing multiple sources, such as authentic compound libraries and tandem mass spectral libraries."
- [intro] empCpd concept: "We have introduced the concept of empirical compound to describe the measurement of a tentative metabolite"
- [intro] MoNA as MS2 database: "and MS2 spectral databases such as MoNA [49] or authentic standards libraries."
