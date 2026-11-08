---
name: ion-mobility-separation-profile-matching
description: Use when LC-IMS-MS/MS data contain MS1 precursor peaks and MS2 fragment
  peaks and fragments must be attributed to precursors by co-elution/co-drift rather
  than retention time alone.
when_to_use_negative:
- Input is a single-dimension (m/z vs RT only) dataset with no drift time/IMS dimension
  — there is no non-m/z separation profile to match, and RT-only matching can falsely
  assign fragments
- Input is already a deconvolved MS2 spectrum or a finalized MS1:MS2 fragment assignment
  table
- The goal is general feature detection or alignment, not tandem MS/MS precursor-fragment
  attribution
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Python API providing the deconvolution module (deimos.deconvolution) whose
    cosine similarity scoring over drift time and retention time profiles implements
    this skill; loaded peak tables come from its feature detection output
  repo: http://github.com/pnnl/deimos
- name: Python
  role: Language in which the DEIMoS API and the deconvolution/deconvolution module
    are executed
- name: LC-IMS-MS/MS
  role: Acquisition modality supplying the multidimensional (m/z, drift time, retention
    time) data whose non-m/z separation profiles are compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:d8e45b21998fe006d68cfac9ded54d10a56c4d68d27568b0141a0dc944591d08
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
  - LC-IMS-MS/MS
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# ion-mobility-separation-profile-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches MS/MS fragment ions to their precursor ions by comparing separation profiles (drift time, retention time) in non-m/z dimensions via cosine similarity, assigning fragments only when profiles co-vary across dimensions.

## When to use

Use when LC-IMS-MS/MS data contain MS1 precursor peaks and MS2 fragment peaks and fragments must be attributed to precursors by co-elution/co-drift rather than retention time alone.

## When NOT to use

- Input is a single-dimension (m/z vs RT only) dataset with no drift time/IMS dimension — there is no non-m/z separation profile to match, and RT-only matching can falsely assign fragments
- Input is already a deconvolved MS2 spectrum or a finalized MS1:MS2 fragment assignment table
- The goal is general feature detection or alignment, not tandem MS/MS precursor-fragment attribution

## Inputs

- MS1 precursor peak table (deimos.load from processed .h5)
- MS2 fragment peak table (deimos.load from processed .h5)
- LC-IMS-MS/MS separation profiles in drift time and retention time dimensions
- User-defined similarity score tolerance

## Outputs

- Retained MS1:MS2 pairing similarity score table
- Deconvolved MS2 fragment assignments per precursor

## How to apply

Load the MS1 and MS2 peak tables from the processed HDF5 data with deimos.load, e.g. keys ms1_peaks and ms2. For each candidate precursor-fragment pairing, extract the separation profiles in the non-m/z dimensions (drift time and retention time) from the LC-IMS-MS/MS space. Compute the cosine similarity between each precursor profile and each fragment profile across the joint drift time-retention time space (DEIMoS deimos.deconvolution logic), and retain only pairings whose similarity score exceeds a user-defined tolerance. Retention time alone is insufficient: in the reported example, masses between 77 and 83 Da would be falsely assigned to the precursor with m/z 212 by RT, whereas the drift time dimension separates them. Using all separation dimensions simultaneously mitigates convolution artifacts in tandem mass spectra and improves attribution confidence.

## Related tools

- **DEIMoS** (Python API providing the deconvolution module (deimos.deconvolution) whose cosine similarity scoring over drift time and retention time profiles implements this skill; loaded peak tables come from its feature detection output) — http://github.com/pnnl/deimos
- **Python** (Language in which the DEIMoS API and the deconvolution/deconvolution module are executed)
- **LC-IMS-MS/MS** (Acquisition modality supplying the multidimensional (m/z, drift time, retention time) data whose non-m/z separation profiles are compared)

## Examples

```
from deimos.deconvolution import compute_cosine_similarity; scores = compute_cosine_similarity(ms1, ms2)
```

## Evaluation signals

- Output score table exists, is parseable, and contains similarity scores for precursor-fragment pairings
- All retained MS1:MS2 pairs have cosine similarity scores exceeding the specified tolerance
- Fragments that would be misassigned by retention time alone (e.g. the 77-83 Da masses vs. the m/z 212 precursor) are excluded by the drift time dimension
- Assigned fragments' separation profiles co-vary with the precursor profile across both drift time and retention time
- Deconvolved MS2 spectra show reduced convolution artifacts compared to RT-only attribution

## Limitations

- Retention time alone yields false MS1:MS2 assignments; multidimensional (drift time plus retention time) scoring is required
- Requires LC-IMS-MS/MS data with at least two separation dimensions beyond m/z — the method depends on drift time resolution to discriminate fragments
- Similarity tolerance is user-defined; an inappropriate cutoff can either drop true fragments or admit false ones
- Relies on prior DEIMoS feature detection and thresholding of the MS1/MS2 peak tables

## Evidence
