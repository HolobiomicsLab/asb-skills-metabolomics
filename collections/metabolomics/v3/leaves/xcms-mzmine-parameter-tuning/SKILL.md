---
name: xcms-mzmine-parameter-tuning
description: Use when comparing how consistently XCMS and MZmine detect and correspond
  LC-MS features on the same centroided mzML dataset, e.g. reproducing the HZV029
  discrepancy (10,901 XCMS vs 42,099 MZmine features, ~60% matched).
when_to_use_negative:
- The goal is downstream annotation or pathway analysis of an existing feature table
  (use khipu/asari annotate or mummichog instead) rather than cross-tool correspondence
  assessment.
- Only one tool's output is needed and no tool-vs-tool comparison is intended — the
  discrepancy quantification is the whole point of this skill.
- Data are low-resolution or from platforms where the processing parameters were never
  established; the reference paper notes settings were developed mostly on Orbitrap
  high-resolution data.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: XCMS
  role: Peak picking and correspondence (cross-sample m/z alignment) to generate the
    first feature table
- name: MZmine
  role: Peak detection and deconvolution modules to generate the second, independently
    defined feature table
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to .mzML so both tools consume identical centroided
    inputs
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: Alternative converter for vendor formats and .mzXML into centroided mzML
  repo: https://proteowizard.sourceforge.io/tools.shtml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:ca72d571b4c5816667949850b39f9fd1029f5c7d1f7c21bc9f719dc66afcff2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compomics/ThermoRawFileParser
  tools:
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - msConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# xcms-mzmine-parameter-tuning

## Summary

A protocol for evaluating cross-tool consistency of LC-MS feature detection and correspondence by running XCMS and MZmine with comparable peak-picking/correspondence parameters on the same converted mzML files and quantifying the resulting feature-count and match-rate discrepancy.

## When to use

Use when comparing how consistently XCMS and MZmine detect and correspond LC-MS features on the same centroided mzML dataset, e.g. reproducing the HZV029 discrepancy (10,901 XCMS vs 42,099 MZmine features, ~60% matched).

## When NOT to use

- The goal is downstream annotation or pathway analysis of an existing feature table (use khipu/asari annotate or mummichog instead) rather than cross-tool correspondence assessment.
- Only one tool's output is needed and no tool-vs-tool comparison is intended — the discrepancy quantification is the whole point of this skill.
- Data are low-resolution or from platforms where the processing parameters were never established; the reference paper notes settings were developed mostly on Orbitrap high-resolution data.

## Inputs

- 184 pooled human plasma HZV029 study LC-MS .RAW files (or any common centroided mzML corpus)
- Centroided mzML files produced via ThermoRawFileParser or msConvert
- Paper-reported peak-picking and correspondence parameters for XCMS
- Paper-reported peak detection and deconvolution module settings for MZmine

## Outputs

- XCMS feature table (feature rows with consensus m/z and retention time)
- MZmine feature table (feature rows from peak detection/deconvolution)
- Summary table of per-tool feature counts and cross-tool match rate
- Validation comparison against reference counts (e.g. 10,901 vs 42,099 features)

## How to apply

Convert the raw Thermo .RAW files to centroided .mzML once, then feed the identical mzML files to both tools so inputs, not conversion settings, differ. Process the dataset in XCMS using the study's peak-picking and correspondence (m/z/RT grouping) parameters to produce a feature table, and in MZmine using the equivalent peak detection and deconvolution modules to produce a second feature table. Count feature rows in each table and record the counts in a summary table — for the 184 pooled-human-plasma HZV029 files the reference values are 10,901 XCMS features and 42,099 MZmine features, with about 60% of XCMS features unambiguously matched to MZmine. Treat moderate count deviations as acceptable software-version drift, but treat large divergence from the ~60% match rate as evidence your parameter choices are not actually 'similar' between tools. The rationale is that the paper attributes such inconsistency to deficiencies in mass alignment and controls of feature quality, so the disagreement itself is the signal being measured.

## Related tools

- **XCMS** (Peak picking and correspondence (cross-sample m/z alignment) to generate the first feature table)
- **MZmine** (Peak detection and deconvolution modules to generate the second, independently defined feature table)
- **ThermoRawFileParser** (Converts Thermo .RAW files to .mzML so both tools consume identical centroided inputs) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (Alternative converter for vendor formats and .mzXML into centroided mzML) — https://proteowizard.sourceforge.io/tools.shtml

## Examples

```
ThermoRawFileParser --input=HZV029_pool.raw --output=mzml  # then run XCMS and MZmine on the identical centroided mzML files and count feature rows in each table
```

## Evaluation signals

- XCMS and MZmine are run on byte-identical centroided mzML files derived from the same RAW conversion step.
- Feature counts land near the paper's reported 10,901 (XCMS) and 42,099 (MZmine) values; minor deviation is acceptable due to software version drift.
- Approximately 60% of XCMS features find unambiguous MZmine counterparts; a substantially higher match rate suggests parameters were tuned beyond 'similar'.
- Both feature tables expose consensus m/z and retention-time columns so correspondence matching (e.g. ppm-based m/z tolerance) can be recomputed.
- A summary table records both counts and the match fraction rather than only qualitative 'differences exist' statements.

## Limitations

- Only ~60% of XCMS features match MZmine under 'similar' parameters; full one-to-one correspondence is not achievable with either tool's defaults.
- Count reproduction is sensitive to software version drift, so exact equality with the published figures is not expected.
- Feature counts differ wildly between tools (here ~4x), so either count alone is a poor proxy for true chemical features — disagreement stems from poor mass alignment and weak feature-quality controls.

## Evidence

- [intro] Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7.: "Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7."
- [abstract] Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [readme] We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML"
- [readme] Msconvert in ProteoWizard (https://proteowizard.sourceforge.io/tools.shtml) can handle the conversion of most vendor data formats and .mzXML files.: "Msconvert in ProteoWizard (https://proteowizard.sourceforge.io/tools.shtml) can handle the conversion of most vendor data formats and .mzXML files."
