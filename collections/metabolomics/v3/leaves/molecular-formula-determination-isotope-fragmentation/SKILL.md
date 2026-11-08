---
name: molecular-formula-determination-isotope-fragmentation
description: Use when high-resolution precursor MS/MS data (mzML/mzXML) must be assigned
  elemental formulas without any spectral or structure database, prior to in silico
  structure candidate ranking.
when_to_use_negative:
- The input is already a feature table or formula-assigned peak list rather than raw
  LC–MS/MS runs
- Only profiled-mode or negative-ion-mode spectra are available — these runs were
  discarded in the published workflow
- The target molecules have mass beyond 700 Da, which the workflow excluded to avoid
  high running time
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS 4
  role: Computes de novo molecular formulas from isotope and fragmentation patterns,
    after feature detection and spectrum merging; processes MassIVE datasets provided
    in mzML or mzXML format
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Consumes the SIRIUS formula ranking to rank structure candidates against a
    structure database (default parameters)
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:1fd9b8d718f7b1fa379f2c78c2de611c94d0a38c23286463d374eb8798c53e24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# molecular-formula-determination-isotope-fragmentation

## Summary

De novo molecular formula determination for LC–MS/MS features using both isotope patterns and fragmentation patterns, as implemented in SIRIUS 4, feeding candidate formulas into downstream structure ranking (CSI:FingerID) and confidence scoring (COSMIC).

## When to use

Use when high-resolution precursor MS/MS data (mzML/mzXML) must be assigned elemental formulas without any spectral or structure database, prior to in silico structure candidate ranking.

## When NOT to use

- The input is already a feature table or formula-assigned peak list rather than raw LC–MS/MS runs
- Only profiled-mode or negative-ion-mode spectra are available — these runs were discarded in the published workflow
- The target molecules have mass beyond 700 Da, which the workflow excluded to avoid high running time

## Inputs

- LC–MS/MS runs in mzML or mzXML format (e.g., 278 runs of MassIVE MSV000082973 mice fecal samples)
- Precursor m/z, adduct, and isotope peak information extracted by SIRIUS feature detection
- High-resolution isotope patterns of the precursor ion

## Outputs

- Ranked de novo molecular formula candidates per feature
- Fragmentation trees with fragment formula annotations and fragmentation events
- Formula-constrained structure candidate lists for CSI:FingerID/COSMIC

## How to apply

Process the LC–MS/MS runs with SIRIUS 4: first collect all fragmentation spectra with their precursor information, search for features associated with those spectra (precursor, adduct, isotope peaks), and merge fragmentation spectra assigned to the same precursor ion using agglomerative clustering based on cosine distance. Then determine the molecular formula de novo using both isotope patterns and fragmentation patterns, with default SIRIUS 4 parameters and a mass accuracy of 10 ppm. Compounds with mass beyond 700 Da should be discarded to avoid high running time, and fragmentation spectra with at most one peak should be removed. The ranked formula output then serves as the constraint for downstream CSI:FingerID structure candidate ranking (e.g., at 40-eV fragmentation or merged spectra) and COSMIC confidence scoring; restrict the candidate list to structures sharing the molecular formula of the top-scoring candidate.

## Related tools

- **SIRIUS 4** (Computes de novo molecular formulas from isotope and fragmentation patterns, after feature detection and spectrum merging; processes MassIVE datasets provided in mzML or mzXML format) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Consumes the SIRIUS formula ranking to rank structure candidates against a structure database (default parameters)) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input MSV000082973_mzml --output sirius_project --maxmz 700 --formula 10ppm
```

## Evaluation signals

- Formulas are determined de novo with 10 ppm mass accuracy — check reported formula masses fall within 10 ppm of the precursor
- Fragmentation spectra with at most one peak must have been removed before formula ranking
- Merged spectra (agglomerative clustering on cosine distance) should yield better annotation rates/separation than individual collision energies, usually followed by 40-eV fragmentation spectra
- Formulas beyond 700 Da must be discarded to avoid high running time
- Downstream CSI:FingerID candidates must share the molecular formula of the top-scoring hit

## Limitations

- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are for academic research and education use only; commercial use requires a Bright Giant license
- The confidence score downstream must not be mistaken as the probability that an annotation is correct; this is impossible by design
- Establishing stereochemistry from fragmentation spectra is beyond automated search engines; only two-dimensional structure is considered
- Small structure modifications are hard and potentially impossible to tell apart using MS/MS data alone

## Evidence

- [discussion] We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm.: "We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm."
- [discussion] Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance.: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] Compounds with mass beyond 700 Da were discarded to avoid high running time.: "Compounds with mass beyond 700 Da were discarded to avoid high running time."
- [readme] SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments: "SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments"
- [readme] SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution.: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
