---
name: spectra-annotation-validation-without-repair
description: Use when you need a counterfactual baseline of how many spectra a require/remove-only
  cleaning pass discards, to quantify the rescue effect of repair functions (e.g.
  repair reduces removals from 83,843 to 31,758 on GNPS).
when_to_use_negative:
- You want the maximum number of usable curated spectra — use the full pipeline with
  repair filters enabled, which removes only 31,758 vs 83,843 spectra
- The goal is to validate only the final cleaned library (448,485 curated spectra)
  rather than to quantify the repair functions' contribution via an ablation run
- Input spectra already lack structural annotations entirely and are intended for
  de novo or embedding-based use, not annotation-based library cleaning
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Provides SpectrumProcessor and the require/remove filter functions used to
    run the no-repair cleaning pass on the GNPS library
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and cross-compares SMILES, InChI and InChIKey annotations so spectra
    with incomplete or non-matching annotations are removed
- name: Python
  role: Language in which the custom SpectrumProcessor configuration and processing-report
    generation are scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:a1c5445f7dc2491f7e4294e215a85e58be45f83c42314006cb54a6a168b0b7b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectra-annotation-validation-without-repair

## Summary

Runs the matchms library-cleaning pipeline on an annotated MS/MS library with all annotation-repair filters disabled, so that spectra failing Require Valid Annotation and parent-mass/annotation consistency checks are removed outright, yielding a baseline removal count for comparison against the with-repair pipeline.

## When to use

Use when you need a counterfactual baseline of how many spectra a require/remove-only cleaning pass discards, to quantify the rescue effect of repair functions (e.g. repair reduces removals from 83,843 to 31,758 on GNPS).

## When NOT to use

- You want the maximum number of usable curated spectra — use the full pipeline with repair filters enabled, which removes only 31,758 vs 83,843 spectra
- The goal is to validate only the final cleaned library (448,485 curated spectra) rather than to quantify the repair functions' contribution via an ablation run
- Input spectra already lack structural annotations entirely and are intended for de novo or embedding-based use, not annotation-based library cleaning

## Inputs

- Raw GNPS public mass spectral library (.msp, 500,569 spectra as downloaded 21-08-2023)
- matchms 0.26.4 Python environment
- Reference with-repair processing report (31,758 spectra removed) for comparison

## Outputs

- Processing report (cond_gnps_no_repair_report.txt) with spectra removed, metadata changed, and peaks changed per filter
- Total spectra-removed count for the no-repair pipeline (83,843)
- Automatically generated YAML file recording all settings and filters used

## How to apply

Load the raw GNPS public mass spectral library (500,569 spectra, downloaded 21-08-2023) into matchms 0.26.4. Build a custom SpectrumProcessor containing only the require/remove filters (e.g. Require Valid Annotation, which loads SMILES/InChI/InChIKey via RDKit and removes spectra whose annotations are incomplete or mismatched), explicitly omitting the repair filters: Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, and Derive Annotation From Compound Name. Execute over all spectra with per-spectrum change logging, then generate a processing report summarizing spectra removed, metadata changed, and peaks changed per filter. Compare the total removals against the with-repair reference count (31,758): the no-repair run removes 83,843 spectra, so the reported count must be strictly greater, demonstrating that repair functions salvage otherwise-discarded spectra (e.g. by deriving SMILES from compound names via PubChem when the parent mass matches).

## Related tools

- **matchms** (Provides SpectrumProcessor and the require/remove filter functions used to run the no-repair cleaning pass on the GNPS library) — https://github.com/matchms/matchms
- **RDKit** (Loads and cross-compares SMILES, InChI and InChIKey annotations so spectra with incomplete or non-matching annotations are removed)
- **Python** (Language in which the custom SpectrumProcessor configuration and processing-report generation are scripted)

## Examples

```
from matchms import SpectrumProcessor; processor = SpectrumProcessor(["require_valid_annotation", "remove_filters_excluding_repairs"]); cleaned = processor.run(load_from_msp("GNPS_library_2023-08-21.msp"))
```

## Evaluation signals

- The spectra-removed count in cond_gnps_no_repair_report.txt is strictly greater than 31,758 (expected 83,843)
- Removed spectra are exactly those lacking a valid annotation or showing a parent mass/annotation mismatch, per the per-filter breakdown in the processing report
- The report confirms none of the five repair filters (Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, Derive Annotation From Compound Name) appear in the applied-filter list
- The auto-generated YAML file lists all settings and filters used, enabling exact reproduction of the ablation configuration
- Input library size before cleaning equals 500,569 spectra, matching the GNPS download of 21-08-2023

## Limitations

- Without Derive Annotation From Compound Name, spectra whose annotations are only compound names (common in GNPS) are removed even when recoverable via PubChem given a matching parent mass
- The pipeline checks metadata consistency only; wrong chemical annotations consistent with the measured mass go unnoticed since no fragment-matching plausibility checks are implemented
- The full GNPS run is compute-intensive (the with-repair pipeline took 6 h and 45 min for 500,569 spectra)

## Evidence

- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] Require Valid Annotation removes mismatched spectra: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] Processing report summarizes per-filter changes: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [results] GNPS library size and download date: "Before cleaning, the GNPS library contained 500,569 spectra."
