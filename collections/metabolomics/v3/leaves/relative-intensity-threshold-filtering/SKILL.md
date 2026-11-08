---
name: relative-intensity-threshold-filtering
description: Use when raw MS/MS spectra from MSP files must be de-noised before model
  input, e.g. dropping peaks with intensity below 1% of the base peak while retaining
  precursor-mass-window consistency.
when_to_use_negative:
- Input spectra are already binarized/merged (ChemEmbed's merged collision-energy
  spectra binarize intensities, so a relative-intensity ranking no longer exists)
- Input is already a filtered peak list or model-ready tensor produced by a previous
  preprocessing run
- The goal is candidate retrieval rather than noise removal — precursor tolerance
  filtering (--precursor_tolerance_ppm) is the relevant mechanism there
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemEmbed
  role: Preprocessing pipeline whose data-curation stage applies the 1%-of-base-peak
    relative-intensity filter to MSP spectra before CNN model input
  repo: github.com/massspecdl/ChemEmbed
- name: pandas
  role: Parses the MSP file into structured DataFrames, applies the per-spectrum intensity
    threshold, and serializes the filtered peak lists to a pickle file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:344f8b9f998366ac1d6aa01876b91a18b5aa6a14e9d18ca9e4280084d4ae7842
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  tools:
  - ChemEmbed
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Relative-intensity threshold filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A per-spectrum noise-filtering step for raw MS/MS spectra that drops any fragment peak whose intensity is below 1% of the spectrum's most intense (base) peak, keeping only signals above the noise floor before model input.

## When to use

Use when raw MS/MS spectra from MSP files must be de-noised before model input, e.g. dropping peaks with intensity below 1% of the base peak while retaining precursor-mass-window consistency.

## When NOT to use

- Input spectra are already binarized/merged (ChemEmbed's merged collision-energy spectra binarize intensities, so a relative-intensity ranking no longer exists)
- Input is already a filtered peak list or model-ready tensor produced by a previous preprocessing run
- The goal is candidate retrieval rather than noise removal — precursor tolerance filtering (--precursor_tolerance_ppm) is the relevant mechanism there

## Inputs

- Raw MS/MS spectra in MSP format (with or without SMILES annotations)
- Per-spectrum precursor m/z and adduct/charge state

## Outputs

- Filtered per-spectrum peak lists (peaks with intensity >= 1% of base peak, and m/z <= precursor + 0.5 Da)
- pandas pickle file of preprocessed spectra (preprocessed_data output of the pipeline)

## How to apply

Parse the input MSP file into a pandas DataFrame that keeps each spectrum's precursor m/z, adduct and peak list. For every spectrum, compute the base-peak (maximum) intensity and drop every peak whose intensity falls below 1% of that value; in the ChemEmbed preprocessing pipeline this relative-intensity threshold is applied together with the mass-window filter that removes peaks exceeding the precursor mass by more than 0.5 Da. The per-spectrum threshold adapts automatically to each spectrum's dynamic range, so no absolute intensity cutoff must be chosen. Serialize the filtered per-spectrum peak lists with pandas to a pickle file as the pipeline's preprocessed_data output. In ChemEmbed, the intensity threshold is exposed as a configurable parameter via the YAML configuration file.

## Related tools

- **ChemEmbed** (Preprocessing pipeline whose data-curation stage applies the 1%-of-base-peak relative-intensity filter to MSP spectra before CNN model input) — github.com/massspecdl/ChemEmbed
- **pandas** (Parses the MSP file into structured DataFrames, applies the per-spectrum intensity threshold, and serializes the filtered peak lists to a pickle file)

## Examples

```
df['peaks'] = df['peaks'].apply(lambda pk: [p for p in pk if p[1] >= 0.01 * max(q[1] for q in pk)])
```

## Evaluation signals

- No retained peak exceeds the precursor m/z by more than 0.5 Da (mass-window invariant)
- Every retained peak intensity is at or above 1% of the spectrum's base-peak intensity
- The base peak itself is retained in every spectrum (its intensity is 100% of the maximum)
- The output pickle contains the same number of spectra as the input MSP file, each with a (possibly shortened) peak list
- Filtering is idempotent: re-running the threshold on filtered spectra removes no additional peaks

## Limitations

- The threshold is relative to each spectrum's base peak, so spectra with overall low signal-to-noise lose proportionally more peaks; the article does not report an absolute floor
- In ChemEmbed the intensity threshold (like resolution and number of top candidates) is user-configurable in the YAML config, so results depend on the chosen value
- Filtering must precede intensity binarization: once merged collision-energy spectra are binarized, relative-intensity information is lost

## Evidence

- [other] 1% of base peak threshold: [evidence span withheld — anchor status: not_found]
