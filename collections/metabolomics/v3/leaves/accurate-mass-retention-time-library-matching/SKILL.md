---
name: accurate-mass-retention-time-library-matching
description: Use when a feature/peak table from HILIC-Q Exactive HF or CSH-Q Exactive
  HF runs contains accurate m/z, retention time, and signal intensities for features
  that lack MS/MS spectra, and matching in-house m/z+RT libraries acquired under identical
  conditions are available.
when_to_use_negative:
- The feature has an acquired MS/MS spectrum — then precursor accurate mass plus MS/MS
  matching (or authentic-standard MSI level 1 identification with precursor mass,
  RT, and MS/MS) should be used instead, since it carries higher confidence
- No in-house library acquired under identical chromatographic and mass-spectrometric
  conditions exists for the assay — retention-time windows are only valid under matched
  conditions
- The goal is MSI level 1 identification — m/z and RT alone can never reach level
  1, which requires authentic standards with precursor mass, retention time, and MS/MS
  matching
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: HILIC-Q Exactive HF mass spectrometry
  role: Source instrument whose HILIC urine feature table (accurate m/z, RT, intensities)
    is matched against the same-condition HILIC m/z+RT library
- name: CSH-Q Exactive HF mass spectrometry
  role: Source instrument whose CSH urine feature table (accurate m/z, RT, intensities)
    is matched against the same-condition CSH m/z+RT library
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:8084ef2937a33cdecb35c306ca2d5d7ac9c3b07af2418bf8150e4cba3b834cc4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - HILIC-Q Exactive HF mass spectrometry
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# accurate-mass-retention-time-library-matching

## Summary

Annotates metabolite features as MSI level 2 using only accurate precursor mass (ppm-level) and retention time matched against in-house libraries acquired under identical chromatographic and MS conditions, without any MS/MS spectral matching. In the urine untargeted metabolomics study this extended coverage beyond the 578 MS/MS-annotated compounds by six additional acylcarnitines and 201 further low-abundance compounds.

## When to use

Use when a feature/peak table from HILIC-Q Exactive HF or CSH-Q Exactive HF runs contains accurate m/z, retention time, and signal intensities for features that lack MS/MS spectra, and matching in-house m/z+RT libraries acquired under identical conditions are available.

## When NOT to use

- The feature has an acquired MS/MS spectrum — then precursor accurate mass plus MS/MS matching (or authentic-standard MSI level 1 identification with precursor mass, RT, and MS/MS) should be used instead, since it carries higher confidence
- No in-house library acquired under identical chromatographic and mass-spectrometric conditions exists for the assay — retention-time windows are only valid under matched conditions
- The goal is MSI level 1 identification — m/z and RT alone can never reach level 1, which requires authentic standards with precursor mass, retention time, and MS/MS matching

## Inputs

- Feature/peak table from HILIC-Q Exactive HF untargeted urine metabolomics run containing accurate m/z, retention time, and signal intensities
- Feature/peak table from CSH-Q Exactive HF untargeted urine metabolomics run containing accurate m/z, retention time, and signal intensities
- In-house m/z and retention-time annotation libraries acquired under identical chromatographic and mass-spectrometric conditions for each respective assay

## Outputs

- mz_rt_annotations.csv containing m/z+RT-only annotations tagged with their MSI-confidence rationale (mass and RT only, no spectral confirmation)
- Flagged set of 6 additional acylcarnitines annotated at MSI level 2
- Flagged set of 201 additional low-abundance compounds matching same-condition m/z and RT library entries

## How to apply

Load the urine feature/peak table with accurate m/z, retention time, and intensities from the HILIC-Q Exactive HF and CSH-Q Exactive HF assays, then import the corresponding in-house m/z and RT annotation libraries acquired under identical chromatographic and mass-spectrometric conditions. Match each measured feature to library entries using ppm-level accurate mass plus retention-time windows only, explicitly excluding any MS/MS spectral matching step. Flag matches that fall within assay-specific accurate-mass and RT tolerance windows as MSI level 2 annotations, tagging each with a rationale of 'mass and RT only, no spectral confirmation'. The rationale is that many low-abundance signals never trigger data-dependent MS/MS, so m/z+RT library matching recovers annotations that spectral workflows miss. Write all flagged annotations to mz_rt_annotations.csv; the study's validation counts were exactly 6 additional acylcarnitines and 201 additional low-abundance compounds.

## Related tools

- **HILIC-Q Exactive HF mass spectrometry** (Source instrument whose HILIC urine feature table (accurate m/z, RT, intensities) is matched against the same-condition HILIC m/z+RT library)
- **CSH-Q Exactive HF mass spectrometry** (Source instrument whose CSH urine feature table (accurate m/z, RT, intensities) is matched against the same-condition CSH m/z+RT library)

## Evaluation signals

- Every flagged annotation must fall within the assay-specific accurate-mass (ppm-level) and RT tolerance windows of its corresponding library entry
- Flag counts must equal exactly 6 acylcarnitines and 201 further low-abundance compounds for this urine dataset
- No annotation in the output may have used an MS/MS spectral matching step — verify the MSI-confidence rationale tags read 'mass and RT only, no spectral confirmation'
- Each annotation must be traceable to a library entry acquired under the identical chromatographic/MS conditions of the matching assay (HILIC or CSH)
- Annotations are explicitly tagged MSI level 2, never level 1

## Limitations

- m/z+RT-only annotations cannot be raised above MSI level 2 and remain restricted to accurate mass information and manual curation
- Valid only when library and sample share identical chromatographic and mass-spectrometric conditions, since retention time matching is condition-dependent
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments, so m/z+RT annotations carry no FDR estimate

## Evidence

- [abstract] Most features lack MS/MS, motivating m/z+RT-only annotation: "Over 9,000 unique metabolite signals were detected, of which 42% triggered MS/MS fragmentations in data-dependent mode"
- [abstract] MSI level 1 requires authentic standards with mass, RT, and MS/MS — the ceiling this skill cannot reach: "we identified 175 compounds using authentic standards with precursor mass, retention time, and MS/MS matching"
- [abstract] The two assays whose feature tables and same-condition libraries this skill matches: "two untargeted metabolomic assays, hydrophilic interaction chromatography (HILIC)-Q Exactive HF mass spectrometry and charged surface hybrid (CSH)-Q Exactive HF mass spectrometry"
- [discussion] MSI level 2 annotations in this study rely on accurate mass information and manual curation: "especially if annotated spectra use accurate mass information and manual curation"
