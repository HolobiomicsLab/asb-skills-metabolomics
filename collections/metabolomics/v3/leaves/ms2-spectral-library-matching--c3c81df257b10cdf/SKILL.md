---
name: ms2-spectral-library-matching
description: Use when mapped empirical compounds carry experimental MS2 spectra and
  putative identifications need reference-library support; i.e., after map_ms2 has
  associated empCpds with MS2 peaks and Level 1 (MS1) annotation is insufficient.
when_to_use_negative:
- The dataset lacks MS2 spectra or empCpds have not been through map_ms2 — use MS1
  annotation against authentic compound databases instead
- Identifications must be confirmed at the highest level; Level 2 is only similarity-based
  annotation, not authentic-standards verification
- 'Data is not LC-MS/MS metabolomics (e.g. GC-MS): the pcpfm pipeline notes GC and
  other data types are not yet supported'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: Provides the l2_annotate operation that loads mapped empCpds, invokes matchms,
    and writes updated empCpd JSON
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: MatchMS
  role: Computes spectral similarity between mapped experimental MS2 spectra and .msp
    library entries, with interval tree optimization
- name: MoNA
  role: Default reference LC-MS/MS (Orbitrap) spectral library, split into positive-
    and negative-mode files
- name: MetDataModel
  role: Defines the JSON-compatible, chainable empirical compound data structure that
    carries Level 2 annotations
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:fd75a457b23c8d6d98ec9145ec698a8208e8865eb6d40576fbddfed739b30d87
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# ms2-spectral-library-matching

## Summary

Annotates empirical compounds at Level 2 (MS/MS spectral similarity) by comparing mapped experimental MS2 spectra against reference tandem MS spectral libraries using matchms within the pcpfm pipeline, by default the MoNA Orbitrap libraries.

## When to use

Use when mapped empirical compounds carry experimental MS2 spectra and putative identifications need reference-library support; i.e., after map_ms2 has associated empCpds with MS2 peaks and Level 1 (MS1) annotation is insufficient.

## When NOT to use

- The dataset lacks MS2 spectra or empCpds have not been through map_ms2 — use MS1 annotation against authentic compound databases instead
- Identifications must be confirmed at the highest level; Level 2 is only similarity-based annotation, not authentic-standards verification
- Data is not LC-MS/MS metabolomics (e.g. GC-MS): the pcpfm pipeline notes GC and other data types are not yet supported

## Inputs

- Empirical compound (empCpd) JSON with MS2 spectra mapped to features/empCpds (e.g. moniker 'MS2_mapped')
- Reference tandem MS spectral library files (.msp), separate for positive and negative ionization modes (default: MoNA Orbitrap libraries downloaded via 'pcpfm download_extras')
- Parameters: --ms2_min_peaks (minimum number of peaks required for a spectrum to be searched/matched) and optional --ms2_similarity_metric override

## Outputs

- Updated empCpd JSON (e.g. moniker 'MS2_MS1_annotated') in which empirical compounds carry Level 2 annotations referencing matched library MS2 spectra

## How to apply

Load the MS2_mapped empirical compound JSON produced by the map_ms2 step, which links features/empCpds to their experimental MS2 spectra. Load reference .msp spectral libraries — either user-supplied files or the MoNA Orbitrap databases fetched via 'pcpfm download_extras' — supplying separate files for positive and negative ionization modes via --msp_files_pos and --msp_files_neg. Run the pcpfm l2_annotate operation, which uses matchms to compute spectral similarity between each experimental MS2 spectrum and library entries, with the interval tree optimization described in the paper; spectra must meet the --ms2_min_peaks minimum peak count to be searched and matched, and any --ms2_similarity_metric override is honored. The best matching library entry is attached to each empirical compound as a Level 2 annotation, and the updated empCpd list is written as JSON under a new moniker (e.g. 'MS2_MS1_annotated'). This follows the paper's design where annotation assigns compound identifications to empirical compounds at graded levels of confidence.

## Related tools

- **pcpfm** (Provides the l2_annotate operation that loads mapped empCpds, invokes matchms, and writes updated empCpd JSON) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **MatchMS** (Computes spectral similarity between mapped experimental MS2 spectra and .msp library entries, with interval tree optimization)
- **MoNA** (Default reference LC-MS/MS (Orbitrap) spectral library, split into positive- and negative-mode files)
- **MetDataModel** (Defines the JSON-compatible, chainable empirical compound data structure that carries Level 2 annotations) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm l2_annotate --moniker MS2_mapped --new_moniker MS2_MS1_annotated --msp_files_pos MONA_Orbitrap_pos.msp --msp_files_neg MONA_Orbitrap_neg.msp --ms2_min_peaks 5
```

## Evaluation signals

- The output empCpd JSON exists and parses successfully as an empCpd list under the new moniker
- Every reported Level 2 annotation corresponds to a reference MS2 spectrum match that met the --ms2_min_peaks peak-count requirement
- Every Level 2 annotation satisfied the configured similarity criterion (--ms2_similarity_metric) against a library entry
- Positive-mode and negative-mode spectra were searched against their respective --msp_files_pos / --msp_files_neg library files
- Matches are anchored to the same empirical compound structure whose features were mapped to MS2 spectra, preserving chainability of the empCpd JSON

## Limitations

- Level 2 is similarity-based: an unresolved empirical compound may correspond to any or a mixture of isomers, so library matches do not resolve isomers
- Spectra with fewer peaks than --ms2_min_peaks are neither searched nor matched, so sparse MS2 spectra yield no Level 2 annotation
- Default reference coverage is limited to the downloadable MoNA Orbitrap databases (and other downloadable sources such as HMDB/LMSD); custom databases require user-supplied .msp files
- Downloading third-party annotation sources obliges the user to respect their licenses (e.g. HMDB is not for commercial use)

## Evidence

- [readme] README workflow lists MS2 annotation step: "perform MS2 annotation (matchms) using a custom database (default MoNA)"
- [readme] README describes empCpd JSON as annotation target: "empirical compounds as a JSON file representing putative metabolites that can be annotated with MS1, MS2, or authentic standards"
- [readme] README describes obtaining the MoNA libraries: "Additional files such as the LC-MS/MS databases from MoNA"
- [intro] Article positions MS2 libraries as annotation sources: "Annotation is a major step after preprocessing, utilizing multiple sources, such as authentic compound libraries and tandem mass spectral libraries."
- [intro] Article names MoNA as an MS2 spectral database: "and MS2 spectral databases such as MoNA [49] or authentic standards libraries."
