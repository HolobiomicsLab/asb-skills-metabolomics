---
name: supplementary-table-reproduction
description: Use when a paper's key quantitative result lives in a supplementary table
  or figure (e.g. class-matching score tables built from MIBiG 2.0) and the goal is
  to rebuild it independently from raw public data and verify it against the published
  supplement.
when_to_use_negative:
- You only need to consume NPClassScore scores in an NPLinker run — use the pre-built
  scoring tables in github.com/louwenjjr/mibig_classifications rather than rebuilding
  them from MIBiG JSON
- Your dataset contains metabolites larger than the ~850 Da CANOPUS mass cut-off or
  spectra without CANOPUS/MolNetEnhancer predictions, since structure-based classes
  needed for the pairing counts will be missing
- You need per-strain BGC–MS/MS link scoring (Metcalf cut-off 2.5, NPClassScore cut-off
  0.25) rather than the population-level class-frequency tables themselves
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: MIBiG
  role: Source of the 1926 experimentally validated BGC–product entries (v2.0 JSON)
    from which the class-pair counts are built
  repo: https://mibig.secondarymetabolites.org
- name: antiSMASH
  role: Provides the genome-based BGC class predictions for each MIBiG entry that
    are cross-referenced to MIBiG classes
- name: NPClassifier
  role: Structure-based chemical class ontology retrieved per SMILES through the GNPS
    API for the structure side of the pairing tables
- name: ClassyFire
  role: Alternative structure-based ontology (superclass/class/subclass) retrieved
    via the GNPS API alongside NPClassifier
- name: NPClassScore
  role: 'Downstream consumer: 28 of the 54 reproduced relative-frequency tables define
    the empirical class-matching score'
  repo: https://github.com/NPLinker/nplinker
- name: mibig_classifications
  role: Jupyter notebooks and scripts that retrieve chemical classes and BGC classifications
    and connect them by counting matched class-term pairs
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:e2c062b01267b2af93c4249ab233d4034524f0db1d13eee4a4b833a11107a7bd
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# supplementary-table-reproduction

## Summary

Reproduces the class-matching scoring tables underlying NPClassScore by downloading all MIBiG 2.0 entries as JSON, extracting SMILES and MIBiG (sub)classes, retrieving NPClassifier/ClassyFire classifications via the GNPS API, and converting genome-side vs structure-side class-pair counts into the 54 relative-frequency tables (28 required by NPClassScore) that reproduce Table S1 and Fig. 1b–c.

## When to use

Use when a paper's key quantitative result lives in a supplementary table or figure (e.g. class-matching score tables built from MIBiG 2.0) and the goal is to rebuild it independently from raw public data and verify it against the published supplement.

## When NOT to use

- You only need to consume NPClassScore scores in an NPLinker run — use the pre-built scoring tables in github.com/louwenjjr/mibig_classifications rather than rebuilding them from MIBiG JSON
- Your dataset contains metabolites larger than the ~850 Da CANOPUS mass cut-off or spectra without CANOPUS/MolNetEnhancer predictions, since structure-based classes needed for the pairing counts will be missing
- You need per-strain BGC–MS/MS link scoring (Metcalf cut-off 2.5, NPClassScore cut-off 0.25) rather than the population-level class-frequency tables themselves

## Inputs

- MIBiG 2.0 entries in JSON format downloaded from mibig.secondarymetabolites.org
- SMILES strings for each MIBiG entry's product
- MIBiG superclass/class/subclass annotations per entry
- antiSMASH-predicted BGC classes cross-referenced to MIBiG classes
- ClassyFire and NPClassifier classifications retrieved via the GNPS API

## Outputs

- 54 class-pairing relative-frequency tables (genome-side BGC class × structure-side chemical class)
- Subset of 28 tables used by NPClassScore (reproducing Table S1)
- Class-matching relationships visualised as in Fig. 1b–c

## How to apply

Download every MIBiG 2.0 entry in JSON format from mibig.secondarymetabolites.org and parse each with Python (json/pandas) to extract the SMILES string and the MIBiG superclass/class/subclass annotations. Map antiSMASH-predicted BGC classes for each entry and cross-reference them to the MIBiG classes, then retrieve ClassyFire and NPClassifier structure classifications for each SMILES through the GNPS API. Count occurrences of each genome-side BGC class paired with each structure-side chemical class and normalise the counts to relative frequencies by dividing by the total occurrence of either the genome-based or the structure-based class. Write all 54 class-pairing relative-frequency tables to disk, flagging the 28 tables consumed by NPClassScore, and compare them against Table S1 and the class pairings shown in Fig. 1b–c. Validation requires each of the 54 tables to cover the complete set of genome- and structure-side classes observed across all 1926 MIBiG 2.0 entries, with the 28-table subset matching the methods description; code and scoring tables are available at github.com/louwenjjr/mibig_classifications (Jupyter notebooks plus scripts using rdkit, plotly 4.14.3, and a conda python=3.7.2 environment).

## Related tools

- **MIBiG** (Source of the 1926 experimentally validated BGC–product entries (v2.0 JSON) from which the class-pair counts are built) — https://mibig.secondarymetabolites.org
- **antiSMASH** (Provides the genome-based BGC class predictions for each MIBiG entry that are cross-referenced to MIBiG classes)
- **NPClassifier** (Structure-based chemical class ontology retrieved per SMILES through the GNPS API for the structure side of the pairing tables)
- **ClassyFire** (Alternative structure-based ontology (superclass/class/subclass) retrieved via the GNPS API alongside NPClassifier)
- **NPClassScore** (Downstream consumer: 28 of the 54 reproduced relative-frequency tables define the empirical class-matching score) — https://github.com/NPLinker/nplinker
- **mibig_classifications** (Jupyter notebooks and scripts that retrieve chemical classes and BGC classifications and connect them by counting matched class-term pairs) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
conda create -n myenv python=3.7.2 rdkit && conda activate myenv && pip install jupyter && jupyter-notebook mibig_classifications.ipynb
```

## Evaluation signals

- All 54 produced relative-frequency tables cover the complete set of genome- and structure-side classes observed across the 1926 MIBiG 2.0 entries
- The subset of 28 tables used by NPClassScore matches the class pairings described in the paper's methods
- Reproduced tables numerically agree with Table S1 and the class-matching relationships in Fig. 1b–c
- Each table entry is a normalised relative frequency (counts divided by total occurrence of either the genome-based or the structure-based class), not a raw count
- Every SMILES in the parsed MIBiG JSON set yields a structure-based classification via the GNPS API or is explicitly accounted for

## Limitations

- Scoring tables are only as good as the MIBiG 2.0 snapshot; the NPClassScore scoring tables 'can be further updated' as more validated entries accrue
- 1988 MS/MS spectra across the case-study datasets lacked CANOPUS or MolNetEnhancer predictions, so structure-side classes can be missing for some compounds
- The reproduction depends on external services (GNPS API for ClassyFire/NPClassifier classifications), so results can drift as those ontologies change

## Evidence

- [results] The class-matching scoring tables were built by counting class matches across MIBiG entries and normalising by the total occurrence of either the genome-based or the structure-based class: [evidence span withheld — anchor status: not_found]
- [discussion] All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format."
- [discussion] the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] These two types of classifications are then connected to eachother by counting their interactions, ideally matching certain chemical and BGC classes: [evidence span withheld — anchor status: external_doc]
- [discussion] NPClassScore is implemented in the NPLinker platform, and the NPClassScore scoring tables can be further updated: "NPClassScore is implemented in the NPLinker platform, and the NPClassScore scoring tables can be further updated"
