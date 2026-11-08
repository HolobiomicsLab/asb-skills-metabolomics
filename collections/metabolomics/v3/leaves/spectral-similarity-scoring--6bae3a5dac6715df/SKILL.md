---
name: spectral-similarity-scoring
description: Use when mapped empirical compound JSON (e.g. 'MS2_mapped' moniker) contains
  experimental MS2 spectra and putative metabolites need Level 2 (MS2-similarity)
  annotations from a reference .msp library.
when_to_use_negative:
- The dataset has no MS2 spectra mapped to empirical compounds — Level 2 requires
  experimental tandem MS spectra; use MS1 authentic-compound-library annotation instead
- The input is a raw file (.raw/.mzML) or an unprocessed feature table rather than
  a mapped empCpd JSON — preprocessing, map_ms2, and pre-annotation (khipu) must run
  first
- The reference library is not in .msp format (or MoNA Orbitrap equivalents are not
  available via 'pcpfm download_extras')
edam_operation: http://edamontology.org/operation_3627
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Hosts the l2_annotate operation that loads mapped empCpds, invokes matchms
    against .msp libraries, and writes the annotated empCpd JSON
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: MatchMS
  role: Computes spectral similarity between mapped experimental MS2 spectra and reference
    library entries (with interval tree optimization)
- name: MoNA
  role: Default reference LC-MS/MS Orbitrap spectral database downloaded via 'pcpfm
    download_extras' and searched as .msp files per ionization mode
- name: MetDataModel
  role: Defines the empirical compound data model (JSON-compatible, chainable) that
    carries the Level 2 annotations
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:b81c7cae3f81d18e5ad6d22a50e9c03ebb2276d9af78fb7413748d1adecc1ea2
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
  edam_operation: http://edamontology.org/operation_3627
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-similarity-scoring

## Summary

Annotates empirical compounds at Level 2 by scoring experimental MS2 spectra against reference tandem MS spectral libraries (default MoNA) with matchms in the pcpfm pipeline, attaching the best-matching library entries as similarity-based compound identifications.

## When to use

Use when mapped empirical compound JSON (e.g. 'MS2_mapped' moniker) contains experimental MS2 spectra and putative metabolites need Level 2 (MS2-similarity) annotations from a reference .msp library.

## When NOT to use

- The dataset has no MS2 spectra mapped to empirical compounds — Level 2 requires experimental tandem MS spectra; use MS1 authentic-compound-library annotation instead
- The input is a raw file (.raw/.mzML) or an unprocessed feature table rather than a mapped empCpd JSON — preprocessing, map_ms2, and pre-annotation (khipu) must run first
- The reference library is not in .msp format (or MoNA Orbitrap equivalents are not available via 'pcpfm download_extras')

## Inputs

- Mapped empirical compound JSON (moniker 'MS2_mapped') from the pcpfm map_ms2 step, linking empCpds/features to experimental MS2 spectra
- Reference MS2 spectral library in .msp format, positive mode (--msp_files_pos), e.g. MoNA Orbitrap library
- Reference MS2 spectral library in .msp format, negative mode (--msp_files_neg), e.g. MoNA Orbitrap library

## Outputs

- Updated empirical compound JSON under a new moniker (e.g. 'MS2_MS1_annotated') with Level 2 MS2 annotations attached
- Level 2 annotation entries referencing best-matching library spectra with similarity scores

## How to apply

Load the mapped empCpd JSON produced by the map_ms2 step, which associates features and empirical compounds with their experimental MS2 spectra. Supply reference tandem MS libraries as .msp files separately per ionization mode via --msp_files_pos and --msp_files_neg; the defaults are the MoNA Orbitrap databases fetched by 'pcpfm download_extras'. Use matchms to compute spectral similarity between each experimental MS2 spectrum and library entries, applying the interval tree optimization described in the paper; filter spectra by --ms2_min_peaks so only spectra with enough peaks are searched and matched, and honor any --ms2_similarity_metric override to change the scoring metric. Attach the best-matching library entries to the empirical compounds as Level 2 annotations (similarity to reference MS2 spectra) and write the updated empCpd list under a new moniker (e.g. 'MS2_MS1_annotated') as a JSON file. This operationalizes the MetDataModel principle that 'annotation assigns compound identification to empirical compound, which accommodates different levels or probable scores.'

## Related tools

- **pcpfm** (Hosts the l2_annotate operation that loads mapped empCpds, invokes matchms against .msp libraries, and writes the annotated empCpd JSON) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **MatchMS** (Computes spectral similarity between mapped experimental MS2 spectra and reference library entries (with interval tree optimization))
- **MoNA** (Default reference LC-MS/MS Orbitrap spectral database downloaded via 'pcpfm download_extras' and searched as .msp files per ionization mode)
- **MetDataModel** (Defines the empirical compound data model (JSON-compatible, chainable) that carries the Level 2 annotations) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm l2_annotate --empCpds MS2_mapped.json --output MS2_MS1_annotated.json --msp_files_pos MoNA_pos.msp --msp_files_neg MoNA_neg.msp --ms2_min_peaks 5
```

## Evaluation signals

- The output empCpd JSON exists and parses successfully as JSON
- Every reported Level 2 annotation corresponds to a reference MS2 spectrum match that met the --ms2_min_peaks requirement
- Each match satisfies the configured similarity criterion (--ms2_similarity_metric or matchms default)
- Positive-mode and negative-mode empCpds were searched against the correct per-mode library files (--msp_files_pos / --msp_files_neg)
- The output appears under a new moniker (e.g. 'MS2_MS1_annotated') preserving the chainable empCpd structure

## Limitations

- Annotation quality depends on library coverage — only the MoNA Orbitrap databases (or user-supplied .msp files) are searched, so compounds absent from the library remain unannotated
- Only spectra passing the minimum peak count filter (--ms2_min_peaks) are searched or matched; sparse MS2 spectra are silently excluded
- Level 2 is similarity-based, not confirmed by authentic standards — unresolved isomers may share library spectra
- MoNA/HMDB extras are third-party resources with license terms (HMDB non-commercial); downloading via 'pcpfm download_extras' implies agreement

## Evidence

- [readme] MS2 annotation via matchms against MoNA is a documented pipeline step: "perform MS2 annotation (matchms) using a custom database (default MoNA)"
- [readme] MoNA libraries are fetched with pcpfm download_extras: "Additional files such as the LC-MS/MS databases from MoNA, a JMS-compliant version of the HMDB and LMSD can be download and placed in the correct directory by running:"
- [intro] Annotation uses tandem mass spectral libraries as a source after preprocessing: "Annotation is a major step after preprocessing, utilizing multiple sources, such as authentic compound libraries and tandem mass spectral libraries."
- [intro] MoNA is named as an MS2 spectral database source: "and MS2 spectral databases such as MoNA [49] or authentic standards libraries."
- [readme] Level assignment is a property of the empCpd annotation model: [evidence span withheld — anchor status: external_doc]
