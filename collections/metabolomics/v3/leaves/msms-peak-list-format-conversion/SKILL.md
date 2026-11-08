---
name: msms-peak-list-format-conversion
description: Use when raw MS/MS contest or experiment peak lists (precursor m/z plus
  fragment m/z-intensity pairs, possibly at several collision energies) must be restructured
  into MAGMa hierarchical input files before in silico candidate annotation.
when_to_use_negative:
- The peak data are already in MAGMa hierarchical input format and only need parameter
  tuning (e.g. changing 3 to 5 bond dissociations)
- The task is downstream candidate retrieval from the local PubChem database or penalty-score
  ranking rather than input preparation
- Annotation should rely on reference-spectrum library matching rather than in silico
  substructure explanation of peaks
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MAGMa
  role: Defines the target hierarchical input format for multistage MS fragmentation
    data and consumes the converted files in command-line mode for substructure-based
    annotation
  repo: https://github.com/NLeSC/MAGMa
- name: emetabolomics web interface
  role: Alternative execution channel that accepts the converted MAGMa input for running
    annotation jobs
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:a9cd60bb201a499454b8965edc9e801f07691ee2af7537fcfdfe676baaac3a0a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-peak-list-format-conversion

## Summary

Converts contest-supplied MS/MS peak lists into MAGMa's hierarchical input format, in which the precursor mono-isotopic peak is followed by its fragment peaks nested in parentheses, optionally merged across collision energies into composite spectra. This formatting step is the prerequisite for MAGMa's substructure-based penalty-score annotation.

## When to use

Use when raw MS/MS contest or experiment peak lists (precursor m/z plus fragment m/z-intensity pairs, possibly at several collision energies) must be restructured into MAGMa hierarchical input files before in silico candidate annotation.

## When NOT to use

- The peak data are already in MAGMa hierarchical input format and only need parameter tuning (e.g. changing 3 to 5 bond dissociations)
- The task is downstream candidate retrieval from the local PubChem database or penalty-score ranking rather than input preparation
- Annotation should rely on reference-spectrum library matching rather than in silico substructure explanation of peaks

## Inputs

- CASMI contest MS/MS peak lists (per challenge: precursor peak plus fragment m/z–intensity pairs)
- MS/MS spectra acquired at different collision energies (to be merged into composite spectra)
- Precursor ion identity/mass (e.g. [M+H]+ or [M-H]- assignment) to anchor the hierarchical input

## Outputs

- MAGMa hierarchical input files (precursor mono-isotopic peak with fragment peaks nested in parentheses)
- Composite spectra merging MS/MS peaks from different collision energies

## How to apply

Take the peak lists provided by the contest (e.g. CASMI 2013 challenges) and convert each into a MAGMa input file that extends the common peak-list format: the precursor mono-isotopic peak is listed first, with the MS/MS fragment peaks nested inside parentheses after it. When a challenge provides spectra at multiple collision energies, construct a composite spectrum by merging the MS/MS peaks, taking the highest intensity for ions present at both energies. Preserve exact m/z values (accurate-mass data) so downstream candidate retrieval can apply a 10 ppm mass deviation with an absolute minimum of 0.002 Da. The converted files are then consumed by MAGMa run in command-line mode or via the emetabolomics web interface; correct nesting of the fragment hierarchy is essential because MAGMa's substructure matching resolves peaks against the precursor-fragment tree.

## Related tools

- **MAGMa** (Defines the target hierarchical input format for multistage MS fragmentation data and consumes the converted files in command-line mode for substructure-based annotation) — https://github.com/NLeSC/MAGMa
- **emetabolomics web interface** (Alternative execution channel that accepts the converted MAGMa input for running annotation jobs)

## Evaluation signals

- Each output file lists the precursor mono-isotopic peak first, with fragment peaks nested in parentheses after it
- m/z values are preserved exactly so a 10 ppm mass deviation with an absolute 0.002 Da minimum remains applicable downstream
- For multi-collision-energy challenges, composite spectra contain each ion once with the highest intensity observed across energies
- The converted files are accepted without error by MAGMa in command-line mode or the emetabolomics web interface and yield candidate rankings consistent with reported outcomes (e.g. correct structure ranked first in 6 of 12 category-2 solutions)

## Limitations

- Conversion does not guarantee peak explanation: with up to 3 bond dissociations several higher-energy peaks in challenge 16 (e.g. m/z=122.0408, 178.03072, 221.07309) remained unexplained, requiring extended fragmentation of up to 5 bond dissociations
- Overly strict mass assumptions carried into interpretation (10 ppm and 0.002 Da) were judged too restrictive for challenge 11, so conversion must not silently round or clip m/z values

## Evidence

- [results] Composite spectra merge peaks across collision energies taking the highest intensity: [evidence span withheld — anchor status: not_found]
- [methods] MAGMa can be run from the command line or via the web interface on the converted input: [evidence span withheld — anchor status: not_found]
