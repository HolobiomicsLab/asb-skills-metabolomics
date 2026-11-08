---
name: lc-ms-retention-time-alignment
description: Use when aligned LC-MS feature tables across many runs show retention
  time drift (e.g., tens of seconds in RPLC negative mode) and features must be mapped
  to a common RT scale before statistical analysis or phenotype prediction.
when_to_use_negative:
- Input is already a fully aligned feature table or you only have a single run/chromatogram
  with no inter-sample RT drift to correct
- The dataset lacks QC pool samples containing unique, high-quality model peaks needed
  as anchors
- The goal is metabolite identification — MassCube's alignment module does not directly
  perform metabolite identification
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MassCube
  role: 'Python framework that performs chromatographic alignment: selects QC model-peak
    anchors and applies linear interpolation to correct nonlinear RT shifts'
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Implementation language of MassCube; the alignment step is invoked from Python
    and interoperates with Python's numerical libraries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:ec7105559add2f6de45b4d7f4d279703212e2a9a1c5ee55fc94a73b2cf1ebf54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-retention-time-alignment

## Summary

Anchor-based retention time alignment for LC-MS feature tables as implemented in MassCube: high-quality model peaks in QC pools serve as anchors, and linear interpolation corrects nonlinear RT shifts across chromatograms in large-scale studies.

## When to use

Use when aligned LC-MS feature tables across many runs show retention time drift (e.g., tens of seconds in RPLC negative mode) and features must be mapped to a common RT scale before statistical analysis or phenotype prediction.

## When NOT to use

- Input is already a fully aligned feature table or you only have a single run/chromatogram with no inter-sample RT drift to correct
- The dataset lacks QC pool samples containing unique, high-quality model peaks needed as anchors
- The goal is metabolite identification — MassCube's alignment module does not directly perform metabolite identification

## Inputs

- MassCube-processed QC feature table with RT, m/z, height, spacing, and noise metadata per feature
- Raw LC-MS data (mzML or Bruker timsTOF .d) already decoded into MassCube's .mcraw workflow cache

## Outputs

- RT-corrected aligned feature table across all runs
- Reported residual RT shift after correction (e.g., ~1.0 s vs. 40.5 s pre-correction in RPLC negative mode)
- Aligned feature groups consumed by downstream gap filling and statistical analysis

## How to apply

Load the MassCube-processed QC feature table carrying RT, m/z, height, spacing, and noise metadata, then identify unique model peaks in the QC pools to act as anchors — unique, high-quality, high-intensity peaks give a stable RT reference across chromatograms. Filter anchor candidates with m/z spacing = 0.01 and noise < 0.3 as validity criteria. Rank valid anchors by peak height and retain the top 50, then split them into train and test subsets. Fit a linear interpolation model on the train-set anchor RTs and apply it to correct RT shifts in the test set, reporting the residual shift (benchmark: reduction from 40.5 s to 1.0 s in RPLC negative mode). Within the full MassCube pipeline, this chromatographic alignment step sits after adduct/ISF grouping and feeds gap filling and aligned feature grouping, which reload the decoded .mcraw cache instead of reparsing the mzML or Bruker .d source.

## Related tools

- **MassCube** (Python framework that performs chromatographic alignment: selects QC model-peak anchors and applies linear interpolation to correct nonlinear RT shifts) — https://github.com/huaxuyu/masscube
- **Python** (Implementation language of MassCube; the alignment step is invoked from Python and interoperates with Python's numerical libraries)

## Examples

```
pip install masscube
```

## Evaluation signals

- Residual test-set RT shift after correction is substantially reduced versus the uncorrected shift; benchmark is ~1.0 s against a 40.5 s pre-correction shift in RPLC negative mode
- All retained anchors satisfy the validity criteria: m/z spacing = 0.01 and noise < 0.3
- Anchor set is the top 50 valid peaks ranked by height, split into train and test subsets
- Downstream gap filling and aligned feature grouping reload the .mcraw cache rather than reparsing the raw source, confirming the alignment ran inside the MassCube workflow
- Corrected RT values map features across chromatograms onto a common scale without introducing shifted group assignments for isotopes, adducts, or in-source fragments

## Limitations

- Anchor quality depends on the QC pools: the shift correction is only as good as the unique, high-quality model peaks available for interpolation
- MassCube does not directly perform metabolite identification, so aligned features still require annotation or confirmation steps; ISFs whose parent ion is fully fragmented in the source cannot be annotated
- Human-defined peak-shape rules cannot account for all peak-shape scenarios, and a GUI is not available in the current version of MassCube

## Evidence

- [intro] adduct grouping including in-source fragments (ISF), chromatographic alignments: "adduct grouping including in-source fragments (ISF), chromatographic alignments"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing.: "masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
