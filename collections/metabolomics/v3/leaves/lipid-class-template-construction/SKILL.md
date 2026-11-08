---
name: lipid-class-template-construction
description: Use when a discovery lipidomics study targets lipid classes not in a
  prebuilt database such as LipidBlast or Lipid Maps — e.g. glycolipids, lipopeptides,
  sphingomyelin, FAHFA, ornithine lipids — or requires fragmentation patterns for
  unique MSn methods.
when_to_use_negative:
- Target lipid classes are already covered by a prebuilt spectral/lipid database such
  as LipidBlast or the Lipid Maps database — use library matching instead of constructing
  new templates.
- Analysis uses LipiDex 1 (Java, .mgf/.mzXML inputs) rather than LipiDex 2 — the simplified
  template interface and MSn tree-based fragmentation support are LipiDex 2 features.
- Workflow input is a peak table (e.g. Compound Discoverer 3.3 .cdResult files) rather
  than a class definition — use Spectrum Searcher or Peak Finder instead of the Library
  Generator.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: LipiDex 2 Library Generator
  role: Module in which lipid classes are constructed from a base template of headgroup,
    backbone, and up to four variable groups, and in silico MSn tree-based fragmentation
    is configured to build the spectral library.
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Stand-alone executable (LipiDex2.exe, requires .NET 7.0) that hosts the Library
    Generator and passes .csv results files through the pipeline to downstream modules
    such as Spectrum Searcher.
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 1
  role: Legacy predecessor (Java, no longer supported) that also created and managed
    custom in-silico lipid spectral libraries using fragmentation templates; available
    at the legacy repository.
  repo: https://github.com/coongroup/LipiDex
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:ace0c6f870133ab0e496a216c1a2bbbbb6aa7af566c12e5d36c225b3fd64ae81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 1
  - LipiDex 2
  - LipiDex 2 Library Generator
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lipid-class-template-construction

## Summary

Constructing custom lipid class templates in the LipiDex 2 Library Generator by building from a base template of headgroup, backbone, and up to four variable groups, then generating an in silico spectral library with MSn tree-based fragmentation for novel or complex lipid classes.

## When to use

Use when a discovery lipidomics study targets lipid classes not in a prebuilt database such as LipidBlast or Lipid Maps — e.g. glycolipids, lipopeptides, sphingomyelin, FAHFA, ornithine lipids — or requires fragmentation patterns for unique MSn methods.

## When NOT to use

- Target lipid classes are already covered by a prebuilt spectral/lipid database such as LipidBlast or the Lipid Maps database — use library matching instead of constructing new templates.
- Analysis uses LipiDex 1 (Java, .mgf/.mzXML inputs) rather than LipiDex 2 — the simplified template interface and MSn tree-based fragmentation support are LipiDex 2 features.
- Workflow input is a peak table (e.g. Compound Discoverer 3.3 .cdResult files) rather than a class definition — use Spectrum Searcher or Peak Finder instead of the Library Generator.

## Inputs

- Lipid class definition: headgroup, backbone, and up to four variable groups (fatty acids, nested lipid classes, or biopolymers such as peptides or glycans)

## Outputs

- In silico spectral library artifact with MSn tree-based fragmentation patterns, consumed by the Spectrum Searcher module
- Spectral entries for defined lipid classes encoding MS2 and higher-order (MS3+) fragmentation logic

## How to apply

In the LipiDex 2 Library Generator (part of the stand-alone LipiDex2.exe, which requires .NET 7.0), construct the lipid class from a base template of headgroup, backbone, and up to four variable groups; the simplified interface supports classes of any complexity, including glycolipids, lipopeptides, sphingomyelin, FAHFA, and ornithine lipids. Variable groups may be fatty acids, other lipid classes (e.g. cardiolipin represented as two phosphatidic acid groups), or biopolymers such as peptides or glycans. Configure in silico fragmentation with MSn tree-based fragmentation methods so that MS2 and higher-order (MS3+) fragmentation logic is encoded in the library. Rationale: this template-based construction defines the level of structural information a spectral match provides, and the generator must 'create libraries with the fragmentation patterns for unique methods'. Output the spectral library artifact for consumption by the Spectrum Searcher module. Validate that generated entries contain the expected MSn tree-based fragmentation patterns for the defined classes.

## Related tools

- **LipiDex 2 Library Generator** (Module in which lipid classes are constructed from a base template of headgroup, backbone, and up to four variable groups, and in silico MSn tree-based fragmentation is configured to build the spectral library.) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Stand-alone executable (LipiDex2.exe, requires .NET 7.0) that hosts the Library Generator and passes .csv results files through the pipeline to downstream modules such as Spectrum Searcher.) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 1** (Legacy predecessor (Java, no longer supported) that also created and managed custom in-silico lipid spectral libraries using fragmentation templates; available at the legacy repository.) — https://github.com/coongroup/LipiDex

## Examples

```
run LipiDex2.exe; open Library Generator; define headgroup, backbone, and up to four variable groups; enable MSn tree-based fragmentation; generate spectral library .csv for Spectrum Searcher
```

## Evaluation signals

- Generated library contains spectral entries for every defined lipid class built from the headgroup/backbone/variable-group template.
- Each entry encodes MSn tree-based fragmentation patterns, consistent with the Library Generator's capability to 'create libraries with the fragmentation patterns for unique methods'.
- Complexity handling confirmed: templates using up to four variable groups, including nested classes (cardiolipin as two PAs) and biopolymers (peptides, glycans), produce valid library entries.
- Library artifact is consumable downstream: Spectrum Searcher can read the generated .csv library file as part of the LipiDex 2 pipeline.
- LipiDex2.exe runs on the target system, confirming the .NET 7.0 dependency is satisfied.

## Limitations

- LipiDex 2 requires .NET 7.0; the operating system will automatically prompt to install it when LipiDex2.exe is first run.
- LipiDex 2's Lipid Spectral Annotator, Degreaser, and MSn tree-based fragmentation features are new relative to LipiDex 1; LipiDex 1 is legacy and no longer supported.
- All modules are stand-alone and pass .csv results files through the pipeline; Peak Finder additionally requires Compound Discoverer version 3.3 .cdResult files, so downstream use of the generated library depends on module-compliant .csv formats.
- Library Generator templates encode structural resolution of spectral matches; insufficient fragment evidence can downgrade identifications (e.g. plasmenyl downgraded to plasmanyl, or to sum composition if a required fragment is absent) — template design must anticipate these downgrades.

## Evidence

- [readme] Legacy LipiDex 1 feature per README: [evidence span withheld — anchor status: external_doc]
