---
name: peak-intensity-scaling
description: Use when pseudo MS/MS spectra reconstructed from MS1-only data (in-source
  fragments) must be matched against GNPS/MoNA/NIST-style reference MS/MS libraries,
  since in-source fragments skew toward low m/z and low collision energies relative
  to reference spectra.
when_to_use_negative:
- The query spectra are true MS/MS spectra acquired at matching collision energies
  — scaling is designed for low-energy in-source fragments in MS1-only pseudo MS/MS
  data
- You are annotating MS/MS (MS2) data directly — the README specifies the gnps.pkl
  (unscaled) library is used for MS/MS annotation, while k10-scaled libraries are
  for MS1 annotation
- The compound class does not produce in-source fragments in the collected m/z range
  (e.g., guanine m/z 150.0421 and glutamic acid m/z 146.0459 failed for lack of sufficient
  in-source fragments above m/z 100)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms1-id
  role: Python package implementing mass-dependent peak intensity scaling (peak_scale_k=10,
    peak_intensity_power=0.5) when indexing reference MS/MS libraries and during precursor-tolerant
    reverse cosine search of pseudo MS/MS spectra
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:f9b7189f16d64bff34e5e9b98148a9195f96c748da1f176ab441fb5a5db2fba8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - METASPACE
  - ms1-id
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peak-intensity-scaling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A mass-dependent reference MS/MS peak intensity scaling approach (scale each fragment intensity by alpha_i = fragment m/z / precursor m/z raised to k=10, then square-root transform both spectra) that enables matching of low-energy in-source fragments in pseudo MS/MS spectra against reference libraries collected at medium-to-high collision energies.

## When to use

Use when pseudo MS/MS spectra reconstructed from MS1-only data (in-source fragments) must be matched against GNPS/MoNA/NIST-style reference MS/MS libraries, since in-source fragments skew toward low m/z and low collision energies relative to reference spectra.

## When NOT to use

- The query spectra are true MS/MS spectra acquired at matching collision energies — scaling is designed for low-energy in-source fragments in MS1-only pseudo MS/MS data
- You are annotating MS/MS (MS2) data directly — the README specifies the gnps.pkl (unscaled) library is used for MS/MS annotation, while k10-scaled libraries are for MS1 annotation
- The compound class does not produce in-source fragments in the collected m/z range (e.g., guanine m/z 150.0421 and glutamic acid m/z 146.0459 failed for lack of sufficient in-source fragments above m/z 100)

## Inputs

- Indexed reference MS/MS library (GNPS July 17, 2024 indexed version from https://github.com/Philipbear/ms1_id/releases, or an .msp/.mgf library indexed with peak_scale_k=10, peak_intensity_power=0.5)
- Pseudo MS/MS spectra in MGF format reconstructed from MS1-only LC-MS or MS imaging data

## Outputs

- Peak-scaled indexed MS/MS library (e.g., gnps_k10.pkl) for reverse spectral search
- Annotation tables with reverse cosine scores (>0.7), matched peak counts (>=3), and spectral usage (>=0.20 LC-MS / >=0.05 MSI), comparing scaled vs non-scaled conditions

## How to apply

For each reference MS/MS spectrum, scale every fragment intensity by its m/z ratio alpha_i (fragment m/z over precursor m/z) raised to a scaling factor of k=10, then apply a square-root transformation (peak_intensity_power 0.5) to both the scaled reference and the query pseudo MS/MS intensities. In ms1-id this is precomputed by building indexed libraries with 'ms1_id index --peak_scale_k 10 --peak_intensity_power 0.5' (e.g., gnps_k10.pkl) or by using the released indexed GNPS library. The scaled library is then searched with the asymmetric reverse cosine score in the flash entropy framework, in a precursor-tolerant open-search mode where unmatched query peaks (adducts, multimers, co-eluting contaminants) are discarded from score calculation. Rationale: of MoNA reference spectra, <1% were acquired at collision energies ≤5 eV, so low-energy in-source fragments systematically mismatch unscaled medium/high-energy references; the mass-dependent scaling compensates for this energy mismatch.

## Related tools

- **ms1-id** (Python package implementing mass-dependent peak intensity scaling (peak_scale_k=10, peak_intensity_power=0.5) when indexing reference MS/MS libraries and during precursor-tolerant reverse cosine search of pseudo MS/MS spectra) — https://github.com/Philipbear/ms1_id

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- In the standard-pool benchmark, the peak-scaled condition yields more matching scores of >0.7 for ground-truth compounds than the non-scaled condition (validated on 14 known molecules in bile acid and drug standard pools)
- For MS imaging, 5 of 7 spotted standards (ATP, ADP, adenosine, glutamine, glutathione) on mouse liver tissue are annotated under the scaled condition; guanine and glutamic acid are expected to fail
- All reported matches satisfy the cutoffs: reverse cosine score >= 0.7, matched peaks >= 3, spectral usage >= 0.20 (LC-MS) or >= 0.05 (MSI)
- Indexed library files with the _k10 suffix (e.g., gnps_k10.pkl) exist and are searched alongside the unscaled library during MS1 annotation

## Limitations

- Scaling cannot compensate for compounds that simply do not generate sufficient in-source fragment ions within the collected m/z range (guanine, glutamic acid failures)
- Reference spectra acquired at low collision energy are extremely scarce (<1% of MoNA spectra at ≤5 eV), so the scaling is an approximation of the energy mismatch rather than an exact correction
- For MS imaging, fragments with m/z < 100 are removed (gnps_minmz100 libraries), so low-mass in-source fragments cannot contribute to matches

## Evidence

- [readme] README: build indexed library with k=10 and power 0.5: [evidence span withheld — anchor status: external_doc]
