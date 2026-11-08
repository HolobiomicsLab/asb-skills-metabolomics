---
name: mibig-json-metadata-extraction
description: Use when a BGC–metabolite class-matching or scoring task needs MIBiG
  entry metadata (SMILES, (sub)classes) extracted from per-entry JSON files as the
  basis for counting genome-side vs structure-side class pairings.
when_to_use_negative:
- When the goal is BGC–MS/MS spectrum linking itself rather than building the underlying
  class-matching scoring tables — use the NPClassScore implementation in NPLinker
  instead
- When MIBiG entries lack SMILES or MIBiG (sub)class annotations, since extraction
  of both fields is required to pair genome- and structure-side classes
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: NPClassifier
  role: Provides structure-based chemical classifications (superclass/class/subclass)
    for each MIBiG SMILES via the GNPS API
- name: ClassyFire
  role: Provides hierarchical superclass/class/subclass ontology classifications for
    each MIBiG SMILES via the GNPS API
- name: antiSMASH
  role: Supplies the genome-based BGC classes that are cross-referenced to MIBiG classes
- name: NPLinker
  role: Consumes the subset of 28 scoring tables in the NPClassScore method to filter
    BGC–MS/MS links
  repo: github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:964ca698824186053a56bf9b2aac5675e64cdad8de0caf58e4c44b0fb30daf34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - MIBiG
  - NPClassifier
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mibig-json-metadata-extraction

## Summary

Parses all MIBiG 2.0 entries downloaded as JSON from mibig.secondarymetabolites.org to extract SMILES strings and MIBiG superclass/class/subclass annotations, then pairs them with antiSMASH-predicted BGC classes and ClassyFire/NPClassifier structure classes to build class-matching relative-frequency tables.

## When to use

Use when a BGC–metabolite class-matching or scoring task needs MIBiG entry metadata (SMILES, (sub)classes) extracted from per-entry JSON files as the basis for counting genome-side vs structure-side class pairings.

## When NOT to use

- When the goal is BGC–MS/MS spectrum linking itself rather than building the underlying class-matching scoring tables — use the NPClassScore implementation in NPLinker instead
- When MIBiG entries lack SMILES or MIBiG (sub)class annotations, since extraction of both fields is required to pair genome- and structure-side classes

## Inputs

- MIBiG 2.0 entry JSON files downloaded from mibig.secondarymetabolites.org (1926 entries)
- antiSMASH-predicted BGC classes for each MIBiG entry
- ClassyFire and NPClassifier structure classifications retrieved via the GNPS API for each SMILES string

## Outputs

- 54 class-pairing relative-frequency tables (28 of which are the subset required by NPClassScore)
- Reproduction of Table S1 and the Fig. 1b–c class-matching relationships

## How to apply

Download all MIBiG 2.0 entries as JSON files from mibig.secondarymetabolites.org and parse each entry with Python (json/pandas) to extract the SMILES string and the MIBiG superclass/class/subclass annotations. Map the antiSMASH-predicted BGC class for each entry and cross-reference it to the MIBiG classes, then retrieve ClassyFire and NPClassifier classifications for each SMILES through the GNPS API. Count occurrences of each genome-side BGC class paired with each structure-side chemical class across the 1926 MIBiG 2.0 entries and convert counts to relative frequencies, normalising by the total occurrence of either the genome-based or the structure-based class. Write the 54 class-pairing relative-frequency tables to disk, keeping the 28 tables consumed by NPClassScore clearly identifiable so they reproduce Table S1 and the class-matching relationships of Fig. 1b–c.

## Related tools

- **NPClassifier** (Provides structure-based chemical classifications (superclass/class/subclass) for each MIBiG SMILES via the GNPS API)
- **ClassyFire** (Provides hierarchical superclass/class/subclass ontology classifications for each MIBiG SMILES via the GNPS API)
- **antiSMASH** (Supplies the genome-based BGC classes that are cross-referenced to MIBiG classes)
- **NPLinker** (Consumes the subset of 28 scoring tables in the NPClassScore method to filter BGC–MS/MS links) — github.com/NPLinker/nplinker

## Examples

```
conda create -n myenv python=3.7.2 rdkit && conda activate myenv && pip install jupyter
```

## Evaluation signals

- Each of the 54 tables covers the complete set of genome- and structure-side classes observed across the 1926 MIBiG 2.0 entries
- The subset of 28 tables used by NPClassScore matches the class pairings described in the methods and Fig. 1b–c
- Relative frequencies are normalised by the total occurrence of either the genome-based or the structure-based class, not raw counts
- Extracted SMILES and MIBiG (sub)classes for every downloaded JSON entry parse without missing fields

## Limitations

- Built on MIBiG 2.0 (1926 entries); scoring tables can be further updated as MIBiG grows
- The lack of verified genome-metabolome links remains a bottleneck for validating the derived class-matching scores
- Candidate links without structure-based classes (e.g. spectra lacking CANOPUS or MolNetEnhancer predictions) cannot be scored downstream — across the study's datasets 1988 MS/MS spectra fell into this category

## Evidence

- [discussion] All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] Series of jupyter notebooks and scripts to retrieve chemical classes (ClassyFire/NPClassifier), as well as BGC classifications for MIBiG BGCs and their products.: [evidence span withheld — anchor status: external_doc]
- [readme] Based on the relative counts of each matched pair of class-terms it is assessed how well the pair of class-terms match to eachother.: [evidence span withheld — anchor status: external_doc]
