---
name: acylcarnitine-library-search
description: Use when untargeted HILIC or CSH LC-MS/MS urine (or similar biofluid)
  spectra contain unmatched acylcarnitine-bearing features that fail authentic-standard
  (MSI level 1) and general library (MoNA/NIST17) searches and need acylcarnitine-specific
  MSI level 2 annotation.
when_to_use_negative:
- Spectra that already match the in-house authentic-standard library on precursor
  mass, retention time, and MS/MS — these qualify for MSI level 1 and should not be
  downgraded via CarniBlast
- Compounds that are not acylcarnitines — route them to MoNA/NIST17 or in silico tools
  such as CSI:FingerID/NIST hybrid search for MSI level 3 instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: CarniBlast
  role: Acylcarnitine-specific MS/MS spectral library used to assign MSI level 2 annotations
    by precursor accurate mass and MS/MS matching
- name: NIST17
  role: General spectral library searched before CarniBlast to annotate non-acylcarnitine
    MSI level 2 compounds
- name: HILIC-Q Exactive HF mass spectrometry
  role: Source instrument for the untargeted urine MS/MS peak lists searched against
    CarniBlast
- name: CSH-Q Exactive HF mass spectrometry
  role: Second untargeted assay providing merged MS/MS peak lists for library searching
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:bfa035d57e481f6a556a7b85af59c33386e55d675df5d515fdb162091783f425
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - CarniBlast
  - HILIC-Q Exactive HF mass spectrometry
  - NIST hybrid search
  - NIST17
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# acylcarnitine-library-search

## Summary

Annotates acylcarnitines in untargeted LC-MS/MS metabolomics data by searching MS/MS spectra against the CarniBlast library, a spectral library specifically geared at acylcarnitines, assigning MSI level 2 confidence annotations by precursor accurate mass and MS/MS matching.

## When to use

Use when untargeted HILIC or CSH LC-MS/MS urine (or similar biofluid) spectra contain unmatched acylcarnitine-bearing features that fail authentic-standard (MSI level 1) and general library (MoNA/NIST17) searches and need acylcarnitine-specific MSI level 2 annotation.

## When NOT to use

- Spectra that already match the in-house authentic-standard library on precursor mass, retention time, and MS/MS — these qualify for MSI level 1 and should not be downgraded via CarniBlast
- Compounds that are not acylcarnitines — route them to MoNA/NIST17 or in silico tools such as CSI:FingerID/NIST hybrid search for MSI level 3 instead

## Inputs

- Merged urine MS/MS peak lists from HILIC-Q Exactive HF and CSH-Q Exactive HF untargeted assays
- Spectra already filtered against in-house authentic-standard library (MSI level 1) and MoNA/NIST17 (MSI level 2 general search)
- CarniBlast acylcarnitine spectral library

## Outputs

- MSI level 2 acylcarnitine annotations with precursor accurate mass and MS/MS match support (e.g. 67 novel acylcarnitines in the paper's urine dataset)
- Combined annotation table with per-MSI-level counts (175 level 1; 578 level 2 including 85 acylcarnitines)

## How to apply

First pass the merged MS/MS peak lists (from HILIC-Q Exactive HF and CSH-Q Exactive HF assays) through the in-house authentic-standard library requiring precursor mass, retention time, and MS/MS agreement for MSI level 1, then through MoNA and NIST17 (precursor accurate mass plus MS/MS matching alone) for MSI level 2. Route the remaining unmatched spectra with acylcarnitine-typical fragment patterns (carnitine moiety signatures) to the CarniBlast library, which is purpose-built for acylcarnitines and recovers species that general metabolite libraries miss. Accept CarniBlast hits as MSI level 2 annotations since matching relies on precursor accurate mass and MS/MS spectra alone, without authentic-standard retention time confirmation. In the Blaženović et al. urine dataset this step added 67 novel acylcarnitine annotations at MSI level 2, contributing to a combined 578 level 2 annotations (85 acylcarnitines in total). Finally merge CarniBlast hits with the other level 1 and level 2 results and tally counts per MSI confidence level for reporting.

## Related tools

- **CarniBlast** (Acylcarnitine-specific MS/MS spectral library used to assign MSI level 2 annotations by precursor accurate mass and MS/MS matching)
- **NIST17** (General spectral library searched before CarniBlast to annotate non-acylcarnitine MSI level 2 compounds)
- **HILIC-Q Exactive HF mass spectrometry** (Source instrument for the untargeted urine MS/MS peak lists searched against CarniBlast)
- **CSH-Q Exactive HF mass spectrometry** (Second untargeted assay providing merged MS/MS peak lists for library searching)

## Evaluation signals

- Every CarniBlast hit is supported by agreement in precursor accurate mass and MS/MS fragment ions (MSI level 2 criteria), with no retention-time claim
- Combined level 2 tally reconciles as general-library hits plus CarniBlast hits (in the paper: 480 via MoNA+NIST17 plus 67 via CarniBlast = 578 at MSI level 2)
- Acylcarnitine count within the level 2 set matches expectation (85 acylcarnitines among the 578 in the paper's urine dataset)
- No spectrum annotated at both level 1 and level 2 — level 1 standard-matched spectra were removed before CarniBlast search

## Limitations

- CarniBlast annotations are MSI level 2 only: precursor accurate mass and MS/MS matching alone cannot reach level 1 without authentic-standard retention time confirmation
- The field lacks generally accepted, validated automated calculations of identification confidence levels with false-discovery rate assessments, so level 2 acylcarnitine calls carry unquantified FDR
- Structure assignments from library matching 'are worthy to be validated by acquiring spectra from corresponding authentic chemicals'

## Evidence
