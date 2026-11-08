---
name: tandem-mass-spectrum-counting
description: Use when you need to describe the size of a set of LC-MS/MS acquisitions,
  e.g. reporting the default compareMS2 QC metric or validating that downloaded ProteomeXchange
  files are complete.
when_to_use_negative:
- When the goal is pairwise spectral similarity or phylogenetic distance between datasets
  — use compareMS2's distance metric instead of simple counts.
- When spectra need to be identified or quantified per peptide; counting spectra is
  not a substitute for database or spectral library search.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: The tool whose default quality-control metric is the per-dataset tandem mass
    spectrum count; also consumes the counted spectra for pairwise distance calculations.
  repo: https://github.com/524D/compareMS2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:1dfdc5dec58b274359f2a7a79f1fe3a6d3b73a189cf0a8294dcd4faf5203b683
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tandem-mass-spectrum-counting

## Summary

Count the tandem mass spectra (MS2) in each LC-MS/MS dataset — e.g. by parsing BEGIN IONS records in MGF files — as the default quality-control metric reported for compareMS2 and its PXD034932 test data.

## When to use

Use when you need to describe the size of a set of LC-MS/MS acquisitions, e.g. reporting the default compareMS2 QC metric or validating that downloaded ProteomeXchange files are complete.

## When NOT to use

- When the goal is pairwise spectral similarity or phylogenetic distance between datasets — use compareMS2's distance metric instead of simple counts.
- When spectra need to be identified or quantified per peptide; counting spectra is not a substitute for database or spectral library search.

## Inputs

- MGF files (LC-MS/MS tandem mass spectra) per dataset
- Dataset identifiers (e.g. ProteomeXchange accessions such as PXD034932) linking each file to its study

## Outputs

- Per-dataset tandem mass spectrum counts (integers), one value per LC-MS/MS dataset
- Optional tabulated counts file (e.g. ms2_spectrum_counts.csv) with dataset ID and count columns
- Validation status per dataset: within/outside any published count range

## How to apply

Retrieve the MGF (or mzML) files for each dataset under study; for MGF, parse the file and count every spectrum record (BEGIN IONS entry), which corresponds to the tandem mass spectra the compareMS2 metric reports. Tabulate one count per dataset alongside its identifier and, where a published range exists (e.g. the five serum datasets in PXD034932, 42,629–47,626 spectra each), verify that each per-dataset count falls inside that range; an out-of-range count flags an incomplete or misparsed file. Because compareMS2 reports pairwise-comparison timing and distance on datasets of this size (10 pairwise comparisons of five serum datasets took ~40 min with default parameters), the counts also let you estimate the compute cost before running the full distance-matrix calculation. Note that counting is a per-file property, distinct from the pairwise spectral similarity that compareMS2 itself computes.

## Related tools

- **compareMS2** (The tool whose default quality-control metric is the per-dataset tandem mass spectrum count; also consumes the counted spectra for pairwise distance calculations.) — https://github.com/524D/compareMS2

## Examples

```
python -c "import re; [print(f,len(re.findall(r'BEGIN IONS',open(f).read()))) for f in ['serum_1.mgf','serum_2.mgf','serum_3.mgf','serum_4.mgf','serum_5.mgf']]"
```

## Evaluation signals

- Each per-dataset count falls within any published range for that study (e.g. 42,629–47,626 for the five serum datasets in PXD034932).
- The sum of counted BEGIN IONS records equals the number of spectra reported by the acquisition or converted file, with no records skipped.
- Counts are one per dataset and reproduce the default compareMS2 QC metric when the same files are used.
- Rerunning the count on the same MGF files yields identical numbers (deterministic parse).

## Limitations

- Counts alone ignore spectral quality, charge-state distribution, and mass accuracy — properties the compareMS2 heatmap exposes but the count does not.
- The count depends on the file format and parsing definition of a 'spectrum' (e.g. MGF BEGIN IONS vs. mzML spectrum elements); mixed formats or truncated files can yield misleading totals.
- compareMS2's QC metric is a size measure only; it does not detect batch effects or outliers that require the pairwise distance matrix.

## Evidence

- [results] five serum datasets contain between 42,629 and 47,626 tandem mass spectra each: "each containing between 42,629 and 47,626 tandem mass spectra"
- [readme] count each spectrum record: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [results] default compareMS2 quality-control metric: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [abstract] proteomeXchange dataset identifier: "The data have been deposited to ProteomeXchange with dataset identifier PXD034932."
- [readme] quality control use of counts: [evidence span withheld — anchor status: external_doc]
