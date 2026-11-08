---
name: spectral-library-ground-truth-matching
description: Use when you need verified ground-truth formulas for compounds in a ZODIAC/SIRIUS
  evaluation, i.e., when scoring annotation accuracy against spectral library search
  hits (e.g., NIST17) on datasets like NIST1950, dendroides, tomato, diatoms, or mice
  stool.
when_to_use_negative:
- When a compound's molecular formula is novel and absent from all public structure
  databases (e.g., PubChem and ChemSpider) and spectral libraries, so no library-search
  ground truth can be established
- When you only have structural database search hits instead of spectral library search
  hits — searching an unknown compound with a novel molecular formula in a structure
  database will yield an incorrect hit that often goes unnoticed
- When the goal is adduct annotation rather than molecular formula validation — adduct
  information cannot be reliably deduced from isotope pattern and MS/MS data and should
  be established during preprocessing instead
tools:
- name: SIRIUS
  role: Provides the ZODIAC-scored molecular formula candidates whose annotations
    are matched against library-search ground truth; ZODIAC is implemented inside
    SIRIUS
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS 2.4.0 (modified)
  role: Processes input mzML/mzXML files and discards low-quality MS/MS spectra before
    library search and annotation
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: NIST17 library
  role: Source of spectral library search hits used to establish ground-truth molecular
    formulas
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:34d41cd9dfb2c0fb57e44ab1ca75a61b0f4e3d8f71b63894c0b340d79b71ac31
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - NIST17 library
  - OpenMS 2.4.0 (modified)
  - SIRIUS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-library-ground-truth-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Establishes ground-truth molecular formula annotations for LC-MS/MS compounds by matching SIRIUS/ZODIAC annotations against spectral library search hits that have undergone manual validation, enabling threshold-based accuracy evaluation.

## When to use

Use when you need verified ground-truth formulas for compounds in a ZODIAC/SIRIUS evaluation, i.e., when scoring annotation accuracy against spectral library search hits (e.g., NIST17) on datasets like NIST1950, dendroides, tomato, diatoms, or mice stool.

## When NOT to use

- When a compound's molecular formula is novel and absent from all public structure databases (e.g., PubChem and ChemSpider) and spectral libraries, so no library-search ground truth can be established
- When you only have structural database search hits instead of spectral library search hits — searching an unknown compound with a novel molecular formula in a structure database will yield an incorrect hit that often goes unnoticed
- When the goal is adduct annotation rather than molecular formula validation — adduct information cannot be reliably deduced from isotope pattern and MS/MS data and should be established during preprocessing instead

## Inputs

- Deposited SIRIUS and ZODIAC result files (from https://bio.informatik.uni-jena.de/data/)
- Spectral library search hits with manual validation (e.g., NIST17 library matches)
- mzML/mzXML input files processed with OpenMS for the LC-MS/MS runs

## Outputs

- Per-compound match table of ZODIAC top-ranked molecular formula vs. ground-truth formula
- Threshold-vs-accuracy table (e.g., zodiac_threshold_accuracy.csv with one row per threshold)

## How to apply

Download the deposited SIRIUS and ZODIAC result files from https://bio.informatik.uni-jena.de/data/ and parse the ZODIAC marginal (posterior) score assigned to each compound's top-ranked molecular formula candidate. Establish ground truth for each compound via spectral library search (e.g., against the NIST17 library) followed by manual validation of the matching spectra. Match each compound's ZODIAC top-ranked annotation against its ground-truth formula, marking compounds without a library match as ungrounded. Then sweep ZODIAC score thresholds (e.g., 0.7, 0.8, 0.9, 0.95, 0.99) and compute, per threshold, the fraction of correct annotations among compounds whose score meets the threshold as well as the fraction of compounds retained. Note that library hits can also serve as anchors inside ZODIAC's stochastic model, but per the paper these contribute only a small effect, so the same library hits should be reserved for evaluation rather than tuning.

## Related tools

- **SIRIUS** (Provides the ZODIAC-scored molecular formula candidates whose annotations are matched against library-search ground truth; ZODIAC is implemented inside SIRIUS) — https://github.com/sirius-ms/sirius
- **OpenMS 2.4.0 (modified)** (Processes input mzML/mzXML files and discards low-quality MS/MS spectra before library search and annotation) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **NIST17 library** (Source of spectral library search hits used to establish ground-truth molecular formulas)

## Evaluation signals

- Selecting annotations with a ZODIAC score threshold of 0.9 yields more than 96.5% correct molecular formula annotations
- At the 0.9 threshold, the fraction of compounds kept lies between 52.05% and 88.24% for each dataset
- Every compound's ZODIAC top-ranked formula either matches its validated ground-truth formula or is explicitly flagged as lacking a library match
- The threshold-vs-accuracy CSV contains one row per evaluated threshold (e.g., 0.7, 0.8, 0.9, 0.95, 0.99) with accuracy and coverage columns
- Matched ground-truth formulas are corroborated by manual validation of the underlying spectral library matches, not by library search alone

## Limitations

- Spectral libraries are of limited size, so only a fraction of acquired MS/MS spectra can be annotated and used as ground truth
- There is no distinct difference between the cosine score of ZODIAC's correct and incorrect molecular formula assignments, so cosine score cannot substitute for library-based ground truth
- Complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible

## Evidence

- [discussion] Anchors have only small effect on annotations: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS framework description from README: [evidence span withheld — anchor status: external_doc]
