---
name: noise-peak-injection-calibration
description: Use when you need to degrade reference MS/MS spectra (mzML/mzXML-derived
  peak lists) to match lower biological-sample quality before evaluating structure-annotation
  confidence scores.
when_to_use_negative:
- The real LC–MS/MS run is already low quality — run the confidence-score evaluation
  on the biological spectra directly rather than on artificially distorted reference
  spectra
- You need true positive labels for a biological dataset — the article states evaluations
  were carried out using reference measurements because 'we do not know the correct
  answers for biological datasets'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: COSMIC
  role: Downstream workflow whose confidence-score evaluation consumes the distorted
    spectra at medium and high noise levels
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes the reference LC–MS/MS spectra and provides the fragmentation spectra
    and features from which distorted spectra are derived
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:634e082813a5027127476f5f78380a422f330994015c107730e70e96ef82515c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# noise-peak-injection-calibration

## Summary

Adds medium or high levels of synthetic noise (mass shifts, intensity distortions, injected noise peaks) to reference tandem MS/MS spectra so the COSMIC confidence score can be evaluated under simulated biological-sample spectral quality.

## When to use

Use when you need to degrade reference MS/MS spectra (mzML/mzXML-derived peak lists) to match lower biological-sample quality before evaluating structure-annotation confidence scores.

## When NOT to use

- The real LC–MS/MS run is already low quality — run the confidence-score evaluation on the biological spectra directly rather than on artificially distorted reference spectra
- You need true positive labels for a biological dataset — the article states evaluations were carried out using reference measurements because 'we do not know the correct answers for biological datasets'

## Inputs

- Reference tandem MS/MS spectra as peak lists of m/z and intensity values (from MassBank, GNPS and NIST 2017, MassHunter Forensics/Toxicology PCDL, or HMDB-derived reference data)

## Outputs

- distorted_spectra.mgf containing distorted fragmentation spectra annotated with distortion level (medium/high), calibrated so the median cosine similarity to the original spectra is 0.880 (medium) and 0.714 (high)

## How to apply

Load reference tandem MS/MS spectra as peak lists and build a pooled noise-peak set from peaks across all reference spectra so injected noise peaks retain realistic m/z and intensity distributions. For the medium distortion level, apply global and per-spectrum mass shifts, random intensity variations, and noise-peak injection, then repeat with stronger parameters for the high level. After each distortion pass, compute the cosine similarity between the distorted spectrum and its original (binned peak-intensity vectors) and take the median per level. Iteratively tune the shift, intensity-variation, and noise-count parameters until the median cosine similarity equals 0.880 for the medium level and 0.714 for the high level. Finally, annotate each distorted spectrum with its distortion level and write the results to distorted_spectra.mgf, ensuring no spectrum was reduced to at most one peak.

## Related tools

- **COSMIC** (Downstream workflow whose confidence-score evaluation consumes the distorted spectra at medium and high noise levels) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes the reference LC–MS/MS spectra and provides the fragmentation spectra and features from which distorted spectra are derived) — https://github.com/sirius-ms/sirius

## Examples

```
python distort_spectra.py --input reference_spectra.mgf --output distorted_spectra.mgf --levels medium,high --cosine-targets 0.880,0.714
```

## Evaluation signals

- Median cosine similarity of distorted vs original spectra equals 0.880 for the medium level and 0.714 for the high level (binned peak-intensity vectors)
- No distorted spectrum has been reduced to at most one peak after distortion
- Every distorted spectrum carries an explicit distortion-level annotation (medium or high)
- Pooled noise peaks used for injection come from the reference spectra themselves so their m/z and intensity distributions match real data

## Limitations

- Confidence scores computed on distorted spectra are a proxy for biological-sample quality; the article notes 'we cannot guarantee a similar FDR for structure annotations below, given our inability to accurately estimate FDR'
- Distortion parameters (mass-shift magnitudes, intensity variation ranges, noise-peak counts) are calibrated by iterative tuning rather than derived from a closed-form model, so the exact settings are dataset-dependent
- The medium and high targets (0.880 and 0.714 cosine similarity) are empirical anchors for this reference-data context and may not transfer to other collision energies or instrument platforms

## Evidence

- [methods] The medium/high distortion levels were tuned so the median cosine similarity of a distorted spectrum to its original matched 0.880 and 0.714 respectively.: [evidence span withheld — anchor status: not_found]
- [methods] To simulate the lower quality of biological-sample spectra, medium or high levels of noise were added to the reference fragmentation spectra before evaluation.: [evidence span withheld — anchor status: not_found]
- [discussion] we used a combined dataset from MassBank68, GNPS37 and the NIST 2017 database: "we used a combined dataset from MassBank68, GNPS37 and the NIST 2017 database"
- [discussion] we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
