---
name: lipid-annotation-lipid-maps-search
description: Use when you have a TIC-normalized peak list of m/z values from MALDI
  FT-ICR mass spectrometry imaging (MSI) data and need to assign putative lipid identities
  before regional or differential analysis.
when_to_use_negative:
- Annotating image-guided single-cell (SCMS) spectra lacking spatial information,
  where FDR-controlled annotation methods are difficult to apply per the article's
  discussion
- When m/z values have not been peak-binned/aligned across sections, causing mass
  shifts that would inflate false matches
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: MALDI FT-ICR mass spectrometer
  role: Instrument that generates the high-mass-resolution MSI pixel spectra whose
    m/z peaks are annotated against LIPID MAPS
- name: MEISTER
  role: Framework whose reconstructed and TIC-normalized 3D MSI peak lists are fed
    into the LIPID MAPS annotation step
  repo: https://github.com/richardxie1119/MEISTER
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:9bcce7ee8f4612d811fc7292985213c46892dacc2232fbc386c27161a5a1343c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lipid-annotation-lipid-maps-search

## Summary

Annotates MSI lipid features by matching detected m/z values against the LIPID MAPS experimental and virtual databases with a ±0.005 m/z threshold, prioritizing experimentally validated lipids sorted by ppm error. This converts a peak list into named lipid species (e.g., HexCer, Cer, SM) that can then be linked to anatomical regions.

## When to use

Use when you have a TIC-normalized peak list of m/z values from MALDI FT-ICR mass spectrometry imaging (MSI) data and need to assign putative lipid identities before regional or differential analysis.

## When NOT to use

- Annotating image-guided single-cell (SCMS) spectra lacking spatial information, where FDR-controlled annotation methods are difficult to apply per the article's discussion
- When m/z values have not been peak-binned/aligned across sections, causing mass shifts that would inflate false matches

## Inputs

- TIC-normalized peak list of m/z values from MALDI FT-ICR MSI pixel spectra
- Peak-binned feature table (e.g., 3-ppm binning) consistent across tissue sections

## Outputs

- Lipid feature annotation table mapping each m/z to a LIPID MAPS entry, with mass error (ppm) and experimental/virtual provenance
- Annotated lipid names (e.g., Cer, SM, HexCer) usable for region-specific enrichment analysis

## How to apply

First ensure pixel intensities have been normalized by total ion count (TIC) and peak lists were binned (e.g., 3-ppm increments) so m/z values are comparable across tissue sections. Then search each detected m/z value against the LIPID MAPS experimental and virtual databases with a ±0.005 m/z threshold. Because virtual entries outnumber experimental ones, sort candidate matches by ppm error and prioritize annotations supported by experimentally validated entries; the paper notes that of 1,156 annotated lipid features, 728 matched with <3 ppm error, illustrating the stringency attainable. Retain the matched annotation, the mass error (ppm), and database provenance (experimental vs virtual) so downstream regional enrichment tables can be validated.

## Related tools

- **MALDI FT-ICR mass spectrometer** (Instrument that generates the high-mass-resolution MSI pixel spectra whose m/z peaks are annotated against LIPID MAPS)
- **MEISTER** (Framework whose reconstructed and TIC-normalized 3D MSI peak lists are fed into the LIPID MAPS annotation step) — https://github.com/richardxie1119/MEISTER

## Evaluation signals

- Verify that each annotated m/z has an assigned LIPID MAPS entry and the mass error is within the ±0.005 m/z threshold
- Check that the fraction of features with <3 ppm error is comparable to the reported ~63% (728 of 1,156 lipid features)
- Confirm annotations prioritize experimentally validated entries over virtual ones when both match
- Validate that downstream regional tables (e.g., myelin-sheath lipids such as Cer elevated in corpus callosum) reproduce reported biological trends

## Limitations

- Annotation accuracy depends on mass shift/alignment; falsely cross-annotated features can arise from an inappropriate ppm window or nonlinearities between MSI and SCMS data
- LIPID MAPS annotation alone does not guarantee uniqueness — isobaric/isomeric lipids within the ±0.005 m/z window cannot be distinguished without additional evidence (e.g., MS/MS or spatial patterns)
- The ±0.005 m/z threshold is instrument-specific (FT-ICR); lower-resolution instruments would need a wider tolerance and would accept more false matches

## Evidence

- [methods] we searched the m/z values against LIPID MAPS experimental and virtual databases with a ±0.005 m/z threshold: [evidence span withheld — anchor status: not_found]
- [results] of a total of 1,156 lipid features annotated by LIPID MAPS34, 728 matched with an error <3 parts per million (ppm) error: "of a total of 1,156 lipid features annotated by LIPID MAPS34, 728 matched with an error <3 parts per million (ppm) error"
- [results] Ceramide, a lipid abundant in the myelin sheath around nerve fibers, was found elevated in subcortical white matter such as the corpus callosum and corticofugal pathway: [evidence span withheld — anchor status: not_found]
- [methods] Each pixel was normalized by total ion count (TIC).: "Each pixel was normalized by total ion count (TIC)."
- [methods] The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
