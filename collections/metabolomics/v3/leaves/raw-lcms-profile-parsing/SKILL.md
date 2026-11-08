---
name: raw-lcms-profile-parsing
description: Use when starting from raw LC-MS-based metabolomics data (e.g., vendor
  profile or centroided intensity profiles) and you need to construct the rank-based
  intensity sorted structure and zone of interest before any feature extraction program
  such as MS-DIAL, XCMS, MZmine, or OpenMS runs.
when_to_use_negative:
- Input is already a feature table produced by XCMS CentWave, MS-DIAL, MZmine, or
  OpenMS — this skill operates on the raw intensity profile upstream of feature extraction.
- The goal is to tune software-specific parameters without inspecting the intermediate
  sorted-intensity and zone-of-interest artifacts — skipping the artifact comparison
  abandons the 'white box' verification the paper emphasizes.
- The dataset lacks vendor-supplied raw intensity profiles and only provides processed
  peak lists, since rank-based intensity sorting and zone-of-interest derivation require
  the original signal profile.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Paramounter
  role: R program whose module consumes the raw profile, performs rank-based intensity
    sorting, derives the zone of interest, and writes the intermediate artifacts that
    are then compared against the deposited reference run
  repo: https://github.com/HuanLab/Paramounter
- name: R
  role: language in which the rank-based intensity sorting and zone-of-interest derivation
    module is implemented and executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:d928ef45a40e961ce55c40f9cb429c5ad838125c13eb9b758d14630fc75fdb14
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - Paramounter
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# raw-lcms-profile-parsing

## Summary

Preprocessing step that loads raw LC-MS-based metabolomics data, rank-sorts intensities in descending order, and derives the zone-of-interest structure used by Paramounter to measure universal processing parameters in a 'white box' manner.

## When to use

Use when starting from raw LC-MS-based metabolomics data (e.g., vendor profile or centroided intensity profiles) and you need to construct the rank-based intensity sorted structure and zone of interest before any feature extraction program such as MS-DIAL, XCMS, MZmine, or OpenMS runs.

## When NOT to use

- Input is already a feature table produced by XCMS CentWave, MS-DIAL, MZmine, or OpenMS — this skill operates on the raw intensity profile upstream of feature extraction.
- The goal is to tune software-specific parameters without inspecting the intermediate sorted-intensity and zone-of-interest artifacts — skipping the artifact comparison abandons the 'white box' verification the paper emphasizes.
- The dataset lacks vendor-supplied raw intensity profiles and only provides processed peak lists, since rank-based intensity sorting and zone-of-interest derivation require the original signal profile.

## Inputs

- raw LC-MS-based metabolomics intensity profile for a single sample file (profile or centroided intensities)
- reference sorted-intensity and zone-of-interest artifacts from a deposited HuanLab/Paramounter example run (for comparison)

## Outputs

- rank-based sorted-intensity structure (CSV)
- zone-of-interest structure (TXT/CSV)
- intermediate artifacts compared against the deposited reference run

## How to apply

Load the raw LC-MS intensity profile for a single sample file and extract the vector of signal intensities. Sort these intensity values in descending rank order to build the sorted-intensity structure that Paramounter operates on. Derive the zone-of-interest structure from that rank-ordered profile, which isolates the signal region the universal parameter measurement relies on. Write the sorted-intensity artifact to CSV and the zone-of-interest artifact to TXT/CSV so the intermediate state can be inspected and compared with the deposited reference run. Then execute the corresponding module of the HuanLab/Paramounter R implementation (https://github.com/HuanLab/Paramounter) on the same input and accept the implementation only when the intermediate artifacts match the deposited example run. The rationale, per the paper, is that these two structures are the 'novel concepts of rank-based intensity sorting and zone of interest' that enable automatic measurement of universal parameters — mass tolerance, peak height, peak width, and instrumental shift — directly from raw data in a 'white box'.

## Related tools

- **Paramounter** (R program whose module consumes the raw profile, performs rank-based intensity sorting, derives the zone of interest, and writes the intermediate artifacts that are then compared against the deposited reference run) — https://github.com/HuanLab/Paramounter
- **R** (language in which the rank-based intensity sorting and zone-of-interest derivation module is implemented and executed)

## Examples

```
Rscript Paramounter_step1_profile_parsing.R --input sample.mzML --out-dir ./parsed
```

## Evaluation signals

- Sorted-intensity artifact matches the deposited reference run's CSV output for the same input file (output_matches_reference).
- Zone-of-interest artifact matches the corresponding TXT/CSV reference from the deposited example run.
- The intermediate artifacts are produced before any feature extraction program (MS-DIAL, XCMS, MZmine, OpenMS) is invoked on the sample.
- The derived zone of interest covers only the signal region within the rank-ordered profile, consistent with the structure Paramounter uses for universal parameter measurement.
- Subsequent universal parameter estimation (mass tolerance, peak height, peak width, instrumental shift) runs on the zone-of-interest output without additional preprocessing.

## Limitations

- Paramounter's optimized peak height threshold trades a higher rate of false positive features for more true positives; a higher threshold (e.g., 2X the optimized value) can reduce false positives but the trade-off originates in this upstream parsing stage.
- Some true positive metabolic features with mass differences smaller than the mzdiff mass tolerance (suggested default 0.001 or 0.01) may be removed by mistake downstream; disabling dereplication via a negative mzdiff value is a relevant follow-on decision.
- Validation on 11 datasets covering different sample types, data acquisition modes, and MS vendors confirms robustness, but the intermediate artifacts must still be diffed against the deposited reference for exact reproducibility.

## Evidence

- [abstract] Paramounter's automatic measurement of universal parameters from raw LC-MS-based metabolomics data is made possible through the novel concepts of rank-based intensity sorting and zone of interest.: [evidence span withheld — anchor status: not_found]
