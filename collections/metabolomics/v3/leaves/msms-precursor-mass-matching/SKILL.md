---
name: msms-precursor-mass-matching
description: Use when MS/MS spectra from untargeted metabolomics have no level 1 hit
  from an authentic-standard library and you need MSI level 2 annotations from public
  or in-silico spectral libraries without retention time confirmation.
when_to_use_negative:
- 'When authentic standards exist for a spectrum: precursor mass + retention time
  + MS/MS matching against the in-house standard library must be used first (MSI level
  1), not mass-only level 2 matching.'
- 'When only precursor accurate mass is available without MS/MS spectra: that corresponds
  to MSI level 3 (e.g. LipidBlast m/z lookup), not this precursor-plus-MS/MS level
  2 procedure.'
- 'When the goal is chemical class ontology of the unannotated remainder: ClassyFire/NIST
  hybrid search superclasses belong to the separate MSI level 3 classification skill,
  not precursor-mass level 2 matching.'
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CSI:FingerID
  role: in silico fragmentation and structure ranking for remaining level 2/3 spectra;
    top-ranked results benchmarked against MSI level 1 set
  repo: https://github.com/sirius-ms/sirius
- name: NIST17
  role: spectral library for precursor mass + MS/MS matching at MSI level 2, including
    NIST hybrid search
- name: CarniBlast
  role: novel library specifically geared at acylcarnitines for MSI level 2 acylcarnitine
    annotations
- name: MoNA
  role: public spectral library searched by precursor accurate mass and MS/MS matching
- name: SIRIUS
  role: element prediction using isotope pattern and fragmentation tree computation
    feeding structure annotation
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:2ab6b6ebccd55b9fc5c0b794e790a648379b2167ca4c5284a86725697f3d7f9d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - CSI:FingerID
  - CarniBlast
  - HILIC-Q Exactive HF mass spectrometry
  - MoNA
  - NIST17
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# msms-precursor-mass-matching

## Summary

Annotate MS/MS spectra at MSI level 2 by matching precursor accurate mass and MS/MS fragment ions against MoNA, NIST17, NIST hybrid search, and the CarniBlast acylcarnitine library, yielding 578 level 2 annotations for urine metabolomics data acquired on HILIC- and CSH-Q Exactive HF instruments.

## When to use

Use when MS/MS spectra from untargeted metabolomics have no level 1 hit from an authentic-standard library and you need MSI level 2 annotations from public or in-silico spectral libraries without retention time confirmation.

## When NOT to use

- When authentic standards exist for a spectrum: precursor mass + retention time + MS/MS matching against the in-house standard library must be used first (MSI level 1), not mass-only level 2 matching.
- When only precursor accurate mass is available without MS/MS spectra: that corresponds to MSI level 3 (e.g. LipidBlast m/z lookup), not this precursor-plus-MS/MS level 2 procedure.
- When the goal is chemical class ontology of the unannotated remainder: ClassyFire/NIST hybrid search superclasses belong to the separate MSI level 3 classification skill, not precursor-mass level 2 matching.

## Inputs

- MS/MS peak lists (precursor m/z + fragment ions) from HILIC-Q Exactive HF urine runs
- MS/MS peak lists from CSH-Q Exactive HF lipidomics/urine runs
- MoNA spectral library
- NIST17 spectral library (with NIST hybrid search)
- CarniBlast acylcarnitine library
- Authentic-standard in-house library results used to exclude level 1 hits

## Outputs

- Annotation table with compound name, precursor mass, MS/MS match, MSI level 2 assignment
- CarniBlast-derived acylcarnitine annotations (85 compounds)
- Level 2 count summary (target 578 compounds)

## How to apply

Start from the MS/MS peak lists produced by the two untargeted assays (HILIC-Q Exactive HF and CSH-Q Exactive HF) after level 1 matching against the in-house authentic-standard library has removed compounds with precursor mass, retention time, and MS/MS agreement. Search the remaining spectra against MoNA and NIST17 using precursor accurate mass and MS/MS fragment matching alone to assign MSI level 2 (480 compounds in this urine study), and add 85 acylcarnitines via the CarniBlast library specifically geared at acylcarnitines. Optionally run NIST hybrid search on candidates whose elemental composition is available (e.g. from SIRIUS fragmentation-tree formula prediction) and validate CSI:FingerID top-ranked structures against the MSI level 1 set, whose top-ranked accuracy was only 40%, so treat in-silico hits as needing confirmation. Annotate every accepted hit with MSI level 2 and tally counts per confidence level; combine the 480 library-based and 98 additional level 2/CarniBlast annotations into the merged annotation table.

## Related tools

- **CSI:FingerID** (in silico fragmentation and structure ranking for remaining level 2/3 spectra; top-ranked results benchmarked against MSI level 1 set) — https://github.com/sirius-ms/sirius
- **NIST17** (spectral library for precursor mass + MS/MS matching at MSI level 2, including NIST hybrid search)
- **CarniBlast** (novel library specifically geared at acylcarnitines for MSI level 2 acylcarnitine annotations)
- **MoNA** (public spectral library searched by precursor accurate mass and MS/MS matching)
- **SIRIUS** (element prediction using isotope pattern and fragmentation tree computation feeding structure annotation) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input msms_peaklists --query spectra.mgf --output sirius_results --csi-fingerid --canopus --max-candidates 10
```

## Evaluation signals

- Level 2 tally must equal 578 compounds (480 via MoNA/NIST17 + 98 via CarniBlast or additional matches).
- Every level 1 assignment must require precursor mass, retention time, and MS/MS agreement — no level 1 hits should come from mass-only matching.
- CSI:FingerID top-ranked accuracy when benchmarked against the MSI level 1 identified compounds should approximate the reported 40%.
- Each accepted level 2 annotation must be supported by both precursor accurate mass and matching MS/MS fragment ions, not by precursor mass alone.
- Acylcarnitine annotations (85) must all originate from the CarniBlast library and appear in the combined level 2 table.

## Limitations

- The field currently lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS.
- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals.
- Structure assignments from Sirius/CSI:Finger ID and NIST17 require validation by acquiring spectra from corresponding authentic chemicals.

## Evidence
