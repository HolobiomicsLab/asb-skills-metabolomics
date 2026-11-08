---
name: msn-tree-fragmentation-method-design
description: Use when building an in silico spectral library for novel or complex
  lipid classes (glycolipids, lipopeptides, FAHFA, cardiolipin) or tailoring fragmentation
  rules to a specific MS2/MS3+ acquisition method in LipiDex 2.
when_to_use_negative:
- The library for the lipid class already exists (e.g. LipidBlast or Lipid Maps spectral
  databases cover the class) and no novel class or method-specific fragmentation is
  needed
- Peak tables, .cdResult files, or raw spectra are the input — spectral matching belongs
  to the Spectrum Searcher/Peak Finder modules, not library generation
- Using legacy LipiDex 1 (Java, .mgf/.mzXML), which lacks MSn tree-based fragmentation
  support and the simplified class-design interface
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0607
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Library Generator
  role: Module that builds lipid classes from headgroup/backbone/variable-group templates
    and encodes MSn tree-based in silico fragmentation into the spectral library
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Spectrum Searcher
  role: Downstream consumer of the generated library; matches experimental MS2/MS3+
    spectra against the encoded fragmentation logic
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Standalone executable (LipiDex2.exe, .NET 7.0) hosting the Library Generator;
    no installation required
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 1
  role: Legacy Java predecessor with template-based fragmentation modeling; no longer
    supported but available for comparison
  repo: https://github.com/coongroup/LipiDex
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:150d9d5969aae20d6a057cd9b422ce2f278b7b1d77762963177d46a3cf5a7d8d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 1
  - LipiDex 2
  - LipiDex 2 Library Generator
  - LipiDex 2 Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0607
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# msn-tree-fragmentation-method-design

## Summary

Design lipid classes and encode MSn tree-based in silico fragmentation logic in the LipiDex 2 Library Generator so that downstream Spectrum Searcher matches are reported at the appropriate structural resolution for complex tandem-MS methods.

## When to use

Use when building an in silico spectral library for novel or complex lipid classes (glycolipids, lipopeptides, FAHFA, cardiolipin) or tailoring fragmentation rules to a specific MS2/MS3+ acquisition method in LipiDex 2.

## When NOT to use

- The library for the lipid class already exists (e.g. LipidBlast or Lipid Maps spectral databases cover the class) and no novel class or method-specific fragmentation is needed
- Peak tables, .cdResult files, or raw spectra are the input — spectral matching belongs to the Spectrum Searcher/Peak Finder modules, not library generation
- Using legacy LipiDex 1 (Java, .mgf/.mzXML), which lacks MSn tree-based fragmentation support and the simplified class-design interface

## Inputs

- Lipid class definitions: headgroup, backbone, and up to four variable groups per class
- Fragmentation templates for MSn tree-based in silico fragmentation (MS2 and MS3+)
- Structural resolution levels for spectral matches (e.g. sum composition vs. fatty acyl detail)
- .NET 7.0 runtime environment (installed automatically when LipiDex2.exe runs)

## Outputs

- In silico spectral library with MSn tree-based fragmentation patterns for the defined lipid classes
- Structural resolution annotations defining what each spectral match level represents
- Spectral library artifact consumed by the LipiDex 2 Spectrum Searcher module

## How to apply

Launch the LipiDex 2 Library Generator module (LipiDex2.exe, requires .NET 7.0) and construct each lipid class from a base template of headgroup, backbone, and up to four variable groups; variable groups may be fatty acyls, other lipid classes (e.g. cardiolipin modeled as two PAs), or biopolymers such as peptides or glycans, using the simplified interface for classes of any complexity. Encode in silico fragmentation as MSn tree-based fragmentation methods so MS2 and higher-order (MS3+) fragmentation logic is fully integrated into the library, and define the structural resolution level each spectral match provides (e.g. required-fragment logic downgrades an identification to sum composition when a required fragment is absent, or to plasmanyl when plasmenyl evidence is insufficient). Generate the spectral library output consumed by the Spectrum Searcher module. This design exists because generic library generators cannot create libraries with fragmentation patterns for unique methods or define the level of structural information a spectral match provides.

## Related tools

- **LipiDex 2 Library Generator** (Module that builds lipid classes from headgroup/backbone/variable-group templates and encodes MSn tree-based in silico fragmentation into the spectral library) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Spectrum Searcher** (Downstream consumer of the generated library; matches experimental MS2/MS3+ spectra against the encoded fragmentation logic) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Standalone executable (LipiDex2.exe, .NET 7.0) hosting the Library Generator; no installation required) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 1** (Legacy Java predecessor with template-based fragmentation modeling; no longer supported but available for comparison) — https://github.com/coongroup/LipiDex

## Evaluation signals

- Generated library contains spectral entries for every defined lipid class with MSn tree-based fragmentation patterns, not flat MS2-only rules
- Fragmentation logic is integrated end-to-end: Spectrum Searcher reports identifications at the structural resolution declared in the library
- Required-fragment behavior is correct: absence of a required fragment downgrades the match to sum composition, and insufficient plasmenyl evidence yields a plasmanyl assignment
- Variable-group design reproduces known complex classes correctly (e.g. cardiolipin represented as two phosphatidic acid groups)
- Library loads in Spectrum Searcher without conversion, matching .raw-file-based Thermo workflows

## Limitations

- LipiDex 2 requires .NET 7.0; the OS prompts for installation on first run of LipiDex2.exe
- Legacy LipiDex 1 is no longer supported and lacks MSn tree fragmentation, glycolipid, and lipopeptide support
- Peak Finder requires Compound Discoverer 3.3 .cdResult files; intermediate .csv conversion is avoided but only for that module
- The Wiki documenting Library Generator usage is a work in progress (WiP), so in-app guidance may be limited

## Evidence
