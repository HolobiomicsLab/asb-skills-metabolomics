---
name: npclassscore-scoring-table-generation
description: Use when building or updating the NPClassScore scoring tables needed
  by NPLinker, i.e. you must bridge genome-based BGC classes to structure-based chemical
  classes using experimentally validated MIBiG BGC–product pairs.
when_to_use_negative:
- You only need to score/filter candidate BGC–MS/MS links in a dataset — apply the
  pre-built NPClassScore tables within NPLinker (Metcalf cut-off 2.5, NPClassScore
  cut-off 0.25) rather than regenerating them from MIBiG.
- Your BGC–compound pairs come from a source other than experimentally validated MIBiG
  entries, so the counted class match frequencies would not be empirically grounded.
- Structure-side classifications cannot be obtained (e.g. spectra without CANOPUS
  or MolNetEnhancer predictions), leaving no structure-based class to pair against.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: MIBiG
  role: Source of experimentally validated BGCs and their products; entries downloaded
    as JSON to supply SMILES and (sub)class annotations
  repo: https://mibig.secondarymetabolites.org
- name: antiSMASH
  role: Provides the genome-based BGC class predictions that are cross-referenced
    to MIBiG classes for each entry
- name: ClassyFire
  role: Structure-based hierarchical ontology (superclass/class/subclass) retrieved
    for each SMILES via the GNPS API
- name: NPClassifier
  role: Structure-based natural product classifier retrieved for each SMILES via the
    GNPS API
- name: NPClassScore
  role: Downstream consumer of the 28 generated scoring tables; implemented in the
    NPLinker platform
  repo: https://github.com/NPLinker/nplinker
- name: mibig_classifications
  role: Jupyter notebooks and scripts that retrieve ClassyFire/NPClassifier and BGC
    classifications for MIBiG entries and build the class-matching tables by counting
    interactions
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:c281eb3e90bfa00a781c3be95b9014a4a21d8811288c1d7ef6c716256c38fd89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - MIBiG
  - NPClassScore
  - NPClassifier
  - NPLinker
  - antiSMASH
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# npclassscore-scoring-table-generation

## Summary

Generates the empirical class-matching scoring tables behind NPClassScore by counting genome-side (antiSMASH/MIBiG) BGC class pairings against structure-side (ClassyFire/NPClassifier) chemical classes across MIBiG 2.0 entries and normalising counts to relative frequencies.

## When to use

Use when building or updating the NPClassScore scoring tables needed by NPLinker, i.e. you must bridge genome-based BGC classes to structure-based chemical classes using experimentally validated MIBiG BGC–product pairs.

## When NOT to use

- You only need to score/filter candidate BGC–MS/MS links in a dataset — apply the pre-built NPClassScore tables within NPLinker (Metcalf cut-off 2.5, NPClassScore cut-off 0.25) rather than regenerating them from MIBiG.
- Your BGC–compound pairs come from a source other than experimentally validated MIBiG entries, so the counted class match frequencies would not be empirically grounded.
- Structure-side classifications cannot be obtained (e.g. spectra without CANOPUS or MolNetEnhancer predictions), leaving no structure-based class to pair against.

## Inputs

- MIBiG 2.0 entry JSON files downloaded from mibig.secondarymetabolites.org (1926 entries)
- SMILES strings extracted from MIBiG entries
- MIBiG superclass/class/subclass annotations
- antiSMASH-predicted BGC classes cross-referenced to MIBiG classes
- ClassyFire and NPClassifier chemical compound classifications retrieved via the GNPS API

## Outputs

- 54 class-pairing relative-frequency tables (counts normalised by genome-side or structure-side class totals)
- Identifiable subset of 28 scoring tables consumed by NPClassScore in NPLinker (reproducing Table S1 and the class-matching relationships of Fig. 1b–c)

## How to apply

Download all MIBiG 2.0 entries as JSON from mibig.secondarymetabolites.org and parse each entry (Python json/pandas) to extract the SMILES string and the MIBiG superclass/class/subclass annotations. Map the antiSMASH-predicted BGC classes for each entry and cross-reference them to the MIBiG classes. Retrieve ClassyFire and NPClassifier classifications for each SMILES through the GNPS API. Count occurrences of each genome-side BGC class paired with each structure-side chemical class, then normalise by the total occurrence of either the genome-based or the structure-based class to obtain relative frequencies (as used in Fig. 1c and Table S1). Write all 54 class-pairing relative-frequency tables to disk; the 28 tables required by NPClassScore must be clearly identifiable. Code and scoring tables are maintained at github.com/louwenjjr/mibig_classifications and the scoring is implemented in NPLinker.

## Related tools

- **MIBiG** (Source of experimentally validated BGCs and their products; entries downloaded as JSON to supply SMILES and (sub)class annotations) — https://mibig.secondarymetabolites.org
- **antiSMASH** (Provides the genome-based BGC class predictions that are cross-referenced to MIBiG classes for each entry)
- **ClassyFire** (Structure-based hierarchical ontology (superclass/class/subclass) retrieved for each SMILES via the GNPS API)
- **NPClassifier** (Structure-based natural product classifier retrieved for each SMILES via the GNPS API)
- **NPClassScore** (Downstream consumer of the 28 generated scoring tables; implemented in the NPLinker platform) — https://github.com/NPLinker/nplinker
- **mibig_classifications** (Jupyter notebooks and scripts that retrieve ClassyFire/NPClassifier and BGC classifications for MIBiG entries and build the class-matching tables by counting interactions) — https://github.com/louwenjjr/mibig_classifications

## Evaluation signals

- Each of the 54 tables covers the complete set of genome- and structure-side classes observed across the 1926 MIBiG 2.0 entries.
- The subset of 28 tables used by NPClassScore matches the class pairings described in the methods.
- Table values are relative frequencies: class-pair counts normalised by the total occurrence of either the genome-based or the structure-based class.
- Reproduced tables match Table S1 and the class-matching relationships shown in Fig. 1b–c.
- Every MIBiG entry parsed contributes a SMILES with both an antiSMASH/MIBiG genome-side class and a retrieved ClassyFire/NPClassifier structure-side classification.

## Limitations

- The tables are empirical and derived from MIBiG 2.0; the lack of verified genome-metabolome links remains a bottleneck, so coverage of class pairings is limited to validated BGC–product pairs.
- Structure-side classification retrieval depends on external services (ClassyFire and NPClassifier through the GNPS API); some SMILES may lack predictions.
- The scoring tables can and should be further updated as MIBiG grows, since NPClassScore is implemented in NPLinker with updateable tables.

## Evidence

- [discussion] All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [results] The class-matching scoring tables were built by counting class matches across MIBiG entries and normalising by the total occurrence of either the genome-based or the structure-based class, as used in Fig. 1c and Table S1.: [evidence span withheld — anchor status: not_found]
- [readme] These two types of classifications are then connected to eachother by counting their interactions, ideally matching certain chemical and BGC classes.: [evidence span withheld — anchor status: external_doc]
- [discussion] Code and scoring tables are available at github.com/louwenjjr/mibig_classifications: "Code and scoring tables are available at github.com/louwenjjr/mibig_classifications"
