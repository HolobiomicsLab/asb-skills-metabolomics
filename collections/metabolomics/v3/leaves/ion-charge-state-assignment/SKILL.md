---
name: ion-charge-state-assignment
description: Use when an aligned multi-sample feature table (with m/z, drift time,
  and retention time dimensions) needs isotopic signature detection and charge-state
  annotation, as required before identifications rely on isotopic evidence.
when_to_use_negative:
- Input is a raw instrument acquisition file (e.g. Agilent 6560 IMS-MS dumps) rather
  than an already detected-and-aligned feature table — feature detection and alignment
  must be performed first.
- The isotopic pattern of interest is not the C13 spacing (1.003355 Da); the skill's
  documented search is parameterized around that delta and ≤5 substitutions.
- Only 1 or 2 isotopologue members are present, since the documented first screening
  requires signatures with at least 3 members.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: provides the isotope detection API (deimos.isotopes) that searches the N-dimensional
    feature table for isotopologues given the C13 m/z delta, substitution and charge
    limits, and dimension tolerances; outputs feed charge-state inference
  repo: http://github.com/pnnl/deimos
- name: Python
  role: language in which DEIMoS is delivered as an API and command-line tool for
    the isotope detection and charge-state annotation workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:b2f8dead78d78d4487926babd5a99c415946bec9bab5a272d4ec545b612e6aa6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ion-charge-state-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns ion charge states to detected features by detecting isotopologues (e.g. C13 spacing of 1.003355 Da) in aligned LC-IMS-MS feature tables with DEIMoS's isotope detection, then inferring charge from the observed isotopologue spacing.

## When to use

Use when an aligned multi-sample feature table (with m/z, drift time, and retention time dimensions) needs isotopic signature detection and charge-state annotation, as required before identifications rely on isotopic evidence.

## When NOT to use

- Input is a raw instrument acquisition file (e.g. Agilent 6560 IMS-MS dumps) rather than an already detected-and-aligned feature table — feature detection and alignment must be performed first.
- The isotopic pattern of interest is not the C13 spacing (1.003355 Da); the skill's documented search is parameterized around that delta and ≤5 substitutions.
- Only 1 or 2 isotopologue members are present, since the documented first screening requires signatures with at least 3 members.

## Inputs

- aligned multi-sample feature table stored in an HDF5 file (e.g. example_alignment.h5 with per-sample keys such as 'A'), with m/z, drift time, and retention time coordinates and intensities

## Outputs

- detected isotopologue groups per monoisotopic feature (isotopic_signatures.h5)
- inferred ion charge state for each monoisotopic feature
- annotated isotopic signature assignments retained after the 50 ppm m/z-error and ≥3-member screens

## How to apply

Load the aligned multi-sample feature table with deimos.load from example_alignment.h5 (per-sample keys) and run DEIMoS isotope detection (deimos.isotopes) over the N-dimensional feature coordinates, constraining the search with the expected C13 isotopic m/z delta of 1.003355 Da, a maximum of 5 isotopic substitutions, a maximum charge of 1, and tol=[0.1, 0.7, 0.15] for m/z, drift time, and retention time. Downselect candidate isotopologues by requiring a maximum m/z error of 50 ppm relative to the expected isotopic signature, then screen to keep only isotopic signatures with at least 3 members as the first quality filter. Infer the ion charge state for each monoisotopic feature from the observed isotopologue spacing and annotate each signature accordingly. The rationale is that DEIMoS operates on N-dimensional data, using all dimensions simultaneously so that the isotopologue search is constrained by the same tolerances used in cross-sample alignment rather than m/z alone.

## Related tools

- **DEIMoS** (provides the isotope detection API (deimos.isotopes) that searches the N-dimensional feature table for isotopologues given the C13 m/z delta, substitution and charge limits, and dimension tolerances; outputs feed charge-state inference) — http://github.com/pnnl/deimos
- **Python** (language in which DEIMoS is delivered as an API and command-line tool for the isotope detection and charge-state annotation workflow)

## Examples

```
import deimos; data = deimos.load('example_alignment.h5', key='A'); sig = deimos.isotopes(data, delta=1.003355, max_sequence=5, max_charge=1, tol=[0.1, 0.7, 0.15])
```

## Evaluation signals

- Retained isotopic signatures contain at least 3 isotopologue members after the first screening.
- Every kept signature satisfies the maximum m/z error of 50 ppm relative to the expected isotopic signature.
- Observed spacing between isotopologues is consistent with the C13 delta (1.003355 Da) and yields a single, consistent charge-state assignment per monoisotopic feature.
- The search tolerances (tol=[0.1, 0.7, 0.15]) leave no isotopologue member drifting outside the m/z, drift time, or retention time bounds.
- Output file (isotopic_signatures.h5) groups isotopologues by monoisotopic feature with charge-state annotations present for each.

## Limitations

- The documented workflow constrains the search to a maximum of 5 isotopic substitutions and a maximum charge of 1, so higher-charge or more heavily substituted patterns are outside the demonstrated parameter set.
- The example workflow screens out signatures with fewer than 3 members, so sparse isotopic evidence may be discarded.
- Isotope detection relies on the initial search being constrained by rough m/z tolerance from the expected m/z delta plus drift time and retention time tolerances; if alignment tolerances are misspecified, candidates may be missed.

## Evidence

- [results] C13 isotopologue detection with delta and limits: [evidence span withheld — anchor status: not_found]
