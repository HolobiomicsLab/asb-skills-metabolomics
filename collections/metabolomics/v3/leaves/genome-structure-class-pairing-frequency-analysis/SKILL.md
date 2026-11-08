---
name: genome-structure-class-pairing-frequency-analysis
description: Use when you need to bridge genome-mining BGC class ontologies (antiSMASH/MIBiG)
  with structure-based chemical ontologies (NPClassifier/ClassyFire) to derive empirical
  class-matching score tables for filtering BGC–MS/MS links.
when_to_use_negative:
- When you only need to apply already-published scoring tables to rank candidate BGC–MS/MS
  links — use NPClassScore directly within NPLinker instead of rebuilding the tables
- When candidate links lack structure-based classes (e.g. the 1988 MS/MS spectra without
  CANOPUS or MolNetEnhancer predictions), since NPClassScore filtering discards such
  links
- When working with metabolites beyond the ~850 Da CANOPUS mass cut-off, where structure-based
  class predictions feeding the pairing tables are unreliable
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0091
tools:
- name: MIBiG
  role: Source of experimentally validated BGC–structure pairs (2.0, 1926 entries)
    from which class pairings are counted
  repo: mibig.secondarymetabolites.org
- name: antiSMASH
  role: Provides genome-based BGC class predictions for each MIBiG entry that are
    cross-referenced to MIBiG classes
- name: ClassyFire
  role: Structure-based hierarchical ontology (superclass/class/subclass) retrieved
    per SMILES via the GNPS API
- name: NPClassifier
  role: Structure-based natural-product classification retrieved per SMILES via the
    GNPS API
- name: NPLinker
  role: Platform in which the resulting scoring tables are implemented as the NPClassScore
    filter
  repo: github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:336d4c5044588607cc9acad54dee9b5a0c8ae44bc7c770a60a7a499298091366
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - MIBiG
  - NPClassifier
  - NPLinker
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Genome–structure class pairing frequency analysis

## Summary

Builds empirical class-matching scoring tables (as used by NPClassScore) by counting pairwise co-occurrences of genome-based BGC classes (antiSMASH/MIBiG superclass/class/subclass) and structure-based chemical classes (NPClassifier/ClassyFire) across all 1926 MIBiG 2.0 entries, then normalising counts to relative frequencies.

## When to use

Use when you need to bridge genome-mining BGC class ontologies (antiSMASH/MIBiG) with structure-based chemical ontologies (NPClassifier/ClassyFire) to derive empirical class-matching score tables for filtering BGC–MS/MS links.

## When NOT to use

- When you only need to apply already-published scoring tables to rank candidate BGC–MS/MS links — use NPClassScore directly within NPLinker instead of rebuilding the tables
- When candidate links lack structure-based classes (e.g. the 1988 MS/MS spectra without CANOPUS or MolNetEnhancer predictions), since NPClassScore filtering discards such links
- When working with metabolites beyond the ~850 Da CANOPUS mass cut-off, where structure-based class predictions feeding the pairing tables are unreliable

## Inputs

- MIBiG 2.0 entry JSON files downloaded from mibig.secondarymetabolites.org
- SMILES strings for each MIBiG compound
- MIBiG superclass/class/subclass annotations per entry
- antiSMASH-predicted BGC classes cross-referenced to MIBiG classes

## Outputs

- 54 class-pairing relative-frequency tables (genome-side BGC class vs structure-side chemical class)
- Subset of 28 scoring tables consumed by NPClassScore in NPLinker
- Reproduction of Table S1 and the class-matching relationships of Fig. 1b–c

## How to apply

Download all MIBiG 2.0 entries as JSON files from mibig.secondarymetabolites.org and parse each entry with Python (json/pandas) to extract the SMILES string and the MIBiG superclass/class/subclass annotations. Map the antiSMASH-predicted BGC classes for each entry and cross-reference them to the MIBiG classes on the genome side. Retrieve ClassyFire and NPClassifier classifications for each SMILES through the GNPS API on the structure side. Count occurrences of each genome-side BGC class paired with each structure-side chemical class and convert counts to relative frequencies, normalising by the total occurrence of either the genome-based or the structure-based class (as in Fig. 1c and Table S1). Write the resulting 54 class-pairing relative-frequency tables to disk, of which the 28 tables used by NPClassScore in NPLinker must be clearly identifiable; these tables are subsequently applied with a cut-off of 0.25 after Standardised Metcalf scoring (cut-off 2.5).

## Related tools

- **MIBiG** (Source of experimentally validated BGC–structure pairs (2.0, 1926 entries) from which class pairings are counted) — mibig.secondarymetabolites.org
- **antiSMASH** (Provides genome-based BGC class predictions for each MIBiG entry that are cross-referenced to MIBiG classes)
- **ClassyFire** (Structure-based hierarchical ontology (superclass/class/subclass) retrieved per SMILES via the GNPS API)
- **NPClassifier** (Structure-based natural-product classification retrieved per SMILES via the GNPS API)
- **NPLinker** (Platform in which the resulting scoring tables are implemented as the NPClassScore filter) — github.com/NPLinker/nplinker

## Examples

```
conda create -n myenv python=3.7.2 rdkit && conda activate myenv && pip install jupyter
```

## Evaluation signals

- Each of the 54 tables covers the complete set of genome- and structure-side classes observed across the 1926 MIBiG 2.0 entries
- The subset of 28 tables used by NPClassScore matches the class pairings described in the methods
- Table values are relative frequencies normalised by the total occurrence of either the genome-based or the structure-based class
- Resulting tables reproduce Table S1 and the class-matching relationships shown in Fig. 1b–c
- Retrieved ClassyFire/NPClassifier labels are obtainable for every extracted MIBiG SMILES via the GNPS API

## Limitations

- Scoring tables are bounded by the content of MIBiG 2.0; the lack of verified genome–metabolome links remains a bottleneck for validating such methods
- antiSMASH output was merged from antiSMASH 3 (50 strains) and antiSMASH 6 runs, introducing version inconsistency in genome-side class calls
- Structure-side classes for downstream linking depend on CANOPUS/MolNetEnhancer; 1988 MS/MS spectra across the datasets lacked such predictions

## Evidence

- [readme] mibig_classifications README: [evidence span withheld — anchor status: external_doc]
- [discussion] article discussion: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format."
