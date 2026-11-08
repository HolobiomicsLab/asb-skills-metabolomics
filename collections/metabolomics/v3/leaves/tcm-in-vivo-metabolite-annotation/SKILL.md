---
name: tcm-in-vivo-metabolite-annotation
description: Use when you have a mouse/in vivo feature list and an in vitro prototype
  list (RT + accurate m/z per component) and must annotate prototypes, isomers, and
  metabolites of TCM components sharing a parent nucleus.
when_to_use_negative:
- Input is still raw LC-MS data requiring background subtraction/peak detection —
  apply the PATBS non-targeted identification step first, since MDRB is a post-processing
  technique operating on already-detected components
- There is no in vitro prototype list to bridge against — MTSF-style single-metabolite-vs-database
  matching or metabolomics approaches are the relevant alternatives
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging post-processing tool that performs
    the prototype/isomer annotation and component relationship network construction
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Implementation language in which the MDRB post-processing and prototype–isomer
    matching are executed
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Upstream precise-and-thorough background-subtraction technique that realizes
    non-targeted identification of metabolites before MDRB bridging
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:5bea718b03742f9b7a34e0552fcfd9d1f3ae2b21c3c02a4bb745364d56d58ed0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tcm-in-vivo-metabolite-annotation

## Summary

Annotates in vivo TCM metabolites and links them to in vitro prototypes using the R-based MDRB post-processing technique, which bridges prototype–metabolite relationships on three dimensions of mass spectral data (m/z, MS1/MS2 mass defect, MS2 fragment similarity).

## When to use

Use when you have a mouse/in vivo feature list and an in vitro prototype list (RT + accurate m/z per component) and must annotate prototypes, isomers, and metabolites of TCM components sharing a parent nucleus.

## When NOT to use

- Input is still raw LC-MS data requiring background subtraction/peak detection — apply the PATBS non-targeted identification step first, since MDRB is a post-processing technique operating on already-detected components
- There is no in vitro prototype list to bridge against — MTSF-style single-metabolite-vs-database matching or metabolomics approaches are the relevant alternatives

## Inputs

- In vitro prototype component list (retention time and accurate m/z per component)
- Mouse (in vivo) metabolite feature list with retention time and accurate m/z
- MS2 fragment spectra for prototype and metabolite candidates (for fragment-similarity and mass-defect scoring)

## Outputs

- Annotated per-pair table (e.g. prototype_isomer_matches.csv) with matched prototype identity or isomer flag, m/z error, and RT difference per mouse metabolite–prototype assignment
- Component relationship network bridging in vitro prototypes and in vivo metabolites sharing the same parent nucleus structure

## How to apply

Load the in vitro prototype list and the mouse (in vivo) metabolite feature list, each with retention time and accurate m/z per component, into R. In MDRB step 1, screen every mouse-metabolite–prototype pair on accurate mass, keeping pairs within ±5 ppm m/z error; classify pairs with |ΔRT| ≤ 25 s as prototype matches (same component observed in vivo) and pairs with |ΔRT| > 25 s as candidate isomers (same nominal mass, chromatographically distinct). Then sequentially analyze the remaining dimensions — mass defect of MS1 and MS2 and similarity of MS2 fragments — scoring metabolite–prototype pairs by structural similarity and minimal mass defect variation, exploiting that MS2 fragment mass defects of prototypes and their metabolites fall within a certain range. Annotate each mouse metabolite with its matched prototype identity or isomer flag and record m/z error and ΔRT per assignment, writing results such as prototype_isomer_matches.csv. The bridged assignments feed a component relationship network that supports in vitro–in vivo correlation and translational metabolic pathway mapping.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging post-processing tool that performs the prototype/isomer annotation and component relationship network construction) — https://github.com/933ZhangDD/MDRB
- **R** (Implementation language in which the MDRB post-processing and prototype–isomer matching are executed) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Upstream precise-and-thorough background-subtraction technique that realizes non-targeted identification of metabolites before MDRB bridging)

## Examples

```
matches <- read.csv("prototype_isomer_matches.csv")  # MDRB step 1 output: one row per mouse metabolite–prototype assignment (±5 ppm, ±25 s)
```

## Evaluation signals

- Every assignment carries an m/z error within the ±5 ppm window
- Prototype matches have |ΔRT| ≤ 25 s; isomer calls have |ΔRT| > 25 s
- Output CSV exists in valid format with the required annotation columns (prototype identity/isomer flag, m/z error, RT difference)
- Each metabolite–prototype pair is scored across all three dimensions (m/z, MS1/MS2 mass defect, MS2 fragment similarity) rather than MS2 similarity alone
- Bridged assignments form a component relationship network correlating in vitro and in vivo data

## Limitations

- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy
- No software versions are stated for the MDRB technique or its data-processing platform, and the repository has no changelog
- No specific mass spectrometer model or data acquisition parameters are named
- Identification of in vivo components remains difficult despite MS data post-processing techniques

## Evidence

- [abstract] MDRB is R-based post-processing: "which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique"
- [abstract] MDRB integrates three mass spectral dimensions: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [intro] MDRB established by integrating m/z, MS2 fragment matching and mass defect: "we integrated the mass spectrometry data of m/z, MS2 fragment matching and mass defect of the difference limit, to establish a novel mass dataset relevance bridging (MDRB)"
- [intro] MS2 fragment mass defect range bridges prototypes and metabolites: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging"
- [discussion] MDRB technique for in vivo characterization bridged with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
