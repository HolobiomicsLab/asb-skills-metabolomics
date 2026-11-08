---
name: precursor-tolerant-open-search
description: Use when annotating pseudo MS/MS spectra (MGF) reconstructed from MS1-only
  LC-MS or imaging MS data, where each spectrum contains multiple ion forms (e.g.
  [M+H]+, adducts, multimers) rather than a single true precursor.
when_to_use_negative:
- When the query spectra are genuine MS/MS with a single predefined precursor — the
  precursor-tolerant reverse search is designed for pseudo MS/MS with multiple ion
  forms, not standard MS/MS library matching
- When only molecular formula annotation (MSI level 4) is needed and no structural
  (level 2/3) annotation via MS/MS library matching is required
- When the reference library was acquired exclusively at high collision energies without
  low-energy spectra — peak scaling was validated on low-energy in-source fragments
  and may not align well with hard fragmentation patterns
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3316
tools:
- name: ms1_id
  role: Python package implementing the precursor-tolerant reverse cosine search against
    indexed MS/MS libraries; used to annotate pseudo MS/MS spectra, LC-MS (mzML/mzXML)
    and imaging MS (imzML) data
  repo: https://github.com/Philipbear/ms1_id
- name: FlashEntropySearch
  role: Underlying flash-entropy spectral matching framework that ms1_id modifies
    for reverse cosine scoring
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: MassCube
  role: Used to exclude isotope peaks from pseudo MS/MS generation prior to the precursor-tolerant
    search
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:2cd1838b2a54b0f29f10f97ba3e36b62cc311bcc2c06e1fc2cb38109a0f73294
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
  - FlashEntropySearch
  - MassCube
  - ms1-id
  - ms1_id
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# precursor-tolerant-open-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An asymmetric reverse spectral search against an indexed MS/MS library that does not assume a single predefined precursor; every ion in the pseudo MS/MS spectrum is treated as a potential precursor, and unmatched query peaks are discarded when computing scores. It enables structural annotation of MS1-only LC-MS and imaging data whose pseudo MS/MS spectra carry adducts, multimers, and co-eluting contaminants absent from reference spectra.

## When to use

Use when annotating pseudo MS/MS spectra (MGF) reconstructed from MS1-only LC-MS or imaging MS data, where each spectrum contains multiple ion forms (e.g. [M+H]+, adducts, multimers) rather than a single true precursor.

## When NOT to use

- When the query spectra are genuine MS/MS with a single predefined precursor — the precursor-tolerant reverse search is designed for pseudo MS/MS with multiple ion forms, not standard MS/MS library matching
- When only molecular formula annotation (MSI level 4) is needed and no structural (level 2/3) annotation via MS/MS library matching is required
- When the reference library was acquired exclusively at high collision energies without low-energy spectra — peak scaling was validated on low-energy in-source fragments and may not align well with hard fragmentation patterns

## Inputs

- Pseudo MS/MS spectra in MGF format (e.g. pseudo_msms.mgf), reconstructed from MS1-only LC-MS or imaging MS data
- Indexed MS/MS reference library (e.g. gnps.pkl and gnps_k10.pkl built from GNPS MGF/MSP with peak_scale_k=10 and peak_intensity_power=0.5)

## Outputs

- Annotation table with compound name, precursor m/z, reverse cosine score, matched peak count, and spectral usage (e.g. ms1_annotation_table.csv or TSV files in the input directory)
- Top-1 annotation per unique precursor m/z per pseudo MS/MS spectrum, satisfying score >= 0.7, matched peaks >= 3, spectral usage >= 0.20 (LC-MS) or >= 0.05 (MSI)

## How to apply

Index the MS/MS reference library (e.g. GNPS, downloaded 17 July 2024) with mass-dependent peak intensity scaling: multiply each fragment intensity by its m/z ratio alpha_i (fragment m/z over precursor m/z) to the power k=10, then apply a square-root (peak_intensity_power 0.5) transform to both reference and query intensities. Run the asymmetric reverse cosine search (flash-entropy framework modified for reverse cosine) in a precursor-tolerant open-search manner: the reference spectrum acts as a template and unmatched peaks in the query pseudo MS/MS spectrum are discarded from the score, so co-eluting contaminants and adduct peaks in the query do not penalize the match. Require that the precursor ion of each matching reference hit appears among the m/z values of the query pseudo MS/MS spectrum. Retain only the top-1 hit per unique precursor m/z, then filter with LC-MS cutoffs: minimum reverse cosine score 0.7, minimum matched peaks 3, minimum spectral usage 0.20 (for MS imaging the spectral usage cutoff is 0.05). Link each retained annotation to a metabolic feature via the pseudo MS/MS spectrum's retention time and the annotation's precursor mass.

## Related tools

- **ms1_id** (Python package implementing the precursor-tolerant reverse cosine search against indexed MS/MS libraries; used to annotate pseudo MS/MS spectra, LC-MS (mzML/mzXML) and imaging MS (imzML) data) — https://github.com/Philipbear/ms1_id
- **FlashEntropySearch** (Underlying flash-entropy spectral matching framework that ms1_id modifies for reverse cosine scoring) — https://github.com/YuanyueLi/FlashEntropySearch
- **MassCube** (Used to exclude isotope peaks from pseudo MS/MS generation prior to the precursor-tolerant search)

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- Every reported annotation satisfies the cutoffs: reverse cosine score >= 0.7, matched peaks >= 3, spectral usage >= 0.20 (LC-MS) or >= 0.05 (MSI)
- Only the top-1 hit is retained per unique precursor m/z for each pseudo MS/MS spectrum
- The precursor ion of each matching reference hit appears among the m/z values of the query pseudo MS/MS spectrum
- Reference library was indexed with peak_scale_k=10 and peak_intensity_power=0.5 (square-root transformation applied to both reference and query intensities)
- Applied to the bile acid and drug standard pool, peak scaling yielded more matching scores >0.7 for ground truths than without scaling

## Limitations

- The ms1-id package version number is not stated in the article (only MassCube ver 1.0.11 and GNPS2 workflow version 2024.09.20 are given)
- Expected runtime is ~5-7 min for a single LC-MS file; if longer than 10 min, the README advises increasing --mass_detect_int_tol (default 2e5 for Orbitraps, 5e2 for QTOFs)
- Only <1% of MoNA reference spectra were acquired at collision energies <=5 eV, so low-energy pseudo MS/MS spectra may still mismatch most public reference libraries
- The score equation and mass-dependent peak scaling equation are referenced in the article text but not fully rendered there

## Evidence

- [results] Reverse search accommodates extra query peaks without penalty: [evidence span withheld — anchor status: not_found]
- [readme] Reference library indexing and search parameters: [evidence span withheld — anchor status: external_doc]
- [readme] Pseudo MS/MS annotation invocation: [evidence span withheld — anchor status: external_doc]
