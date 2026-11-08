---
name: msn-tree-fragmentation-annotation
description: Use when building lipid spectral libraries or annotating lipid identifications
  from Thermo raw LC-MSn data where MS3+ spectra and tree-based fragment logic (e.g.
  required-fragment downgrades) are needed for structural confidence.
when_to_use_negative:
- Input is already a filtered peak/feature table with no spectra — use Peak Finder
  QC filtering instead, not library generation or fragment annotation
- Data are non-MSn (MS2-only, non-Thermo) and the user needs no tree-based fragmentation
  or MS3+ annotation logic
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: LipiDex 2 Library Generator
  role: Builds in silico spectral libraries with MSn tree-based fragmentation from
    headgroup/backbone/variable-group templates
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Spectrum Searcher
  role: Matches acquired MS2 and MS3+ spectra against the generated library with RT
    filtering, MSn intensity threshold, and PPM/Thomson mass error parameters
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Lipid Spectral Annotator
  role: Annotates fragment peaks using Spectrum Searcher MS2 and MS3+ spectral matches
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:61b0d58863e3d08da43dc511759731f6e6f0065aa608f41cd65c222e11025649
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 2
  - LipiDex 2 Library Generator
  - LipiDex 2 Lipid Spectral Annotator
  - LipiDex 2 Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# msn-tree-fragmentation-annotation

## Summary

Generate in silico lipid spectral libraries with MSn tree-based fragmentation and annotate MS2/MS3+ fragment peaks, so identifications are made at the appropriate structural resolution using the full MSn fragmentation tree rather than MS2 alone.

## When to use

Use when building lipid spectral libraries or annotating lipid identifications from Thermo raw LC-MSn data where MS3+ spectra and tree-based fragment logic (e.g. required-fragment downgrades) are needed for structural confidence.

## When NOT to use

- Input is already a filtered peak/feature table with no spectra — use Peak Finder QC filtering instead, not library generation or fragment annotation
- Data are non-MSn (MS2-only, non-Thermo) and the user needs no tree-based fragmentation or MS3+ annotation logic

## Inputs

- Thermo .raw LC-MSn files (MS2 and MS3+ spectra)
- In silico lipid spectral library (Library Generator template: headgroup, backbone, up to four variable groups)
- Spectrum Searcher MS2/MS3+ spectral match results (.csv)

## Outputs

- In silico spectral library with MSn tree-based fragmentation patterns
- Fragment-annotated lipid identifications from Lipid Spectral Annotator
- Structurally resolved annotations with required-fragment downgrades (sum composition / plasmanyl fallbacks)

## How to apply

Use the LipiDex 2 Library Generator to build an in silico spectral library from a base template of headgroup, backbone, and up to four variable groups, enabling MSn tree-based fragmentation, glycolipids, and lipopeptides. Run Spectrum Searcher against Thermo .raw files to match MS2 and MS3+ spectra with RT filtering, MSn intensity threshold, and PPM or Thomson mass error parameters. Feed matches to the Lipid Spectral Annotator, which annotates fragment peaks from MS2 and MS3+ matches. Apply tree-based required-fragment logic: if a class-diagnostic required fragment is absent, downgrade the identification to sum composition, and downgrade plasmenyl to plasmanyl when ether evidence is insufficient. Rationale: fully integrating fragmentation logic into the data pipeline ensures identifications carry only the structural resolution the spectra actually support.

## Related tools

- **LipiDex 2 Library Generator** (Builds in silico spectral libraries with MSn tree-based fragmentation from headgroup/backbone/variable-group templates) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Spectrum Searcher** (Matches acquired MS2 and MS3+ spectra against the generated library with RT filtering, MSn intensity threshold, and PPM/Thomson mass error parameters) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Lipid Spectral Annotator** (Annotates fragment peaks using Spectrum Searcher MS2 and MS3+ spectral matches) — https://github.com/coongroup/LipiDex-2/releases/latest

## Examples

```
LipiDex2.exe  # run Library Generator -> Spectrum Searcher -> Lipid Spectral Annotator (stand-alone modules, .NET 7.0)
```

## Evaluation signals

- Spectral matches carry the expected structural resolution: required-fragment absence downgrades identification to sum composition, not a full species call
- Insufficient plasmenyl evidence results in plasmanyl downgrade, checkable by comparing annotated subclass before/after logic application
- Library Generator output contains fragmentation patterns for each designed class built from the headgroup/backbone/variable-group template
- Lipid Spectral Annotator output maps fragment peaks for every MS2 and MS3+ match in the lipid feature
- End-to-end reproducibility check: reprocessing MSV000091033 yields 779 identified features pre-filter (719 post Peak Finder filtering)

## Limitations

- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table, so upstream feature extraction is tied to that tool version
- LipiDex 2 requires .NET 7.0; modules are stand-alone but pass .csv results files through the pipeline (intermediate files required)
- LipiDex 2 Wiki instructions were work-in-progress at the time of the README

## Evidence
