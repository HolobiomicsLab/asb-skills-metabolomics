---
name: mibig-scoring-table-construction
description: 'Use when building or updating NPClassScore lookup tables: you have MIBiG
  2.0 json entries and need genome-based vs structure-based class co-occurrence frequencies
  to score candidate BGC–MS/MS links.'
when_to_use_negative:
- You only need to score/filter already-generated candidate BGC–MS/MS links — use
  the NPClassScore lookup step in NPLinker directly with existing tables instead of
  rebuilding them.
- Your compounds or BGCs cannot be represented in the ClassyFire/NPClassifier and
  antiSMASH/BiG-SCAPE ontologies the tables bridge (e.g. metabolites above the ~850
  Da CANOPUS mass cut-off have no structure-based class predictions to tabulate).
- You need experimentally verified BGC–metabolite links rather than class-level co-occurrence
  statistics — the article notes the lack of verified links remains a validation bottleneck
  and MIBiG provides only a limited number.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MIBiG 2.0
  role: Source of experimentally validated BGCs and their products used to bridge
    genome-based and structure-based classification ontologies
- name: ClassyFire
  role: Hierarchical structure-based chemical ontology (superclass, class, subclass)
    assigned to MIBiG product SMILES
- name: NPClassifier
  role: Natural-product-specific structure-based classification of MIBiG product SMILES,
    taking chemistry and biosynthetic pathways into account
- name: antiSMASH
  role: Genome-based BGC (sub)class assignments for MIBiG entries feeding the genome
    side of the scoring tables
- name: BiG-SCAPE
  role: Groups BGCs and provides genome-based class annotations for the genome side
    of the tables
- name: GNPS API
  role: Retrieves ClassyFire and NPClassifier chemical compound classifications for
    the MIBiG SMILES
  repo: https://github.com/NPLinker/nplinker
- name: mibig_classifications
  role: Notebooks and scripts that retrieve both classification types for MIBiG BGCs
    and count class-term interactions into relative-frequency scoring tables
  repo: github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:8885b67381df78534bbdc2f4283bc11cd8b880284c60364a79ba2b5f60ceff2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - ClassyFire
  - GNPS API
  - MIBiG
  - MIBiG 2.0
  - MolNetEnhancer
  - NPClassifier
  - NPLinker
  - antiSMASH
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# mibig-scoring-table-construction

## Summary

Constructs precomputed scoring tables that map pairs of genome-based BGC classes (from antiSMASH/BiG-SCAPE) to structure-based chemical classes (from ClassyFire/NPClassifier) using MIBiG 2.0 entries, recording relative frequencies of each matched class-pair. These tables are the lookup backbone for NPClassScore filtering of BGC–MS/MS candidate links in NPLinker.

## When to use

Use when building or updating NPClassScore lookup tables: you have MIBiG 2.0 json entries and need genome-based vs structure-based class co-occurrence frequencies to score candidate BGC–MS/MS links.

## When NOT to use

- You only need to score/filter already-generated candidate BGC–MS/MS links — use the NPClassScore lookup step in NPLinker directly with existing tables instead of rebuilding them.
- Your compounds or BGCs cannot be represented in the ClassyFire/NPClassifier and antiSMASH/BiG-SCAPE ontologies the tables bridge (e.g. metabolites above the ~850 Da CANOPUS mass cut-off have no structure-based class predictions to tabulate).
- You need experimentally verified BGC–metabolite links rather than class-level co-occurrence statistics — the article notes the lack of verified links remains a validation bottleneck and MIBiG provides only a limited number.

## Inputs

- MIBiG 2.0 entries downloaded from mibig.secondarymetabolites.org in json format
- SMILES structures extracted from MIBiG entries
- MIBiG BGC (sub)classes
- antiSMASH and BiG-SCAPE BGC classifications for MIBiG entries
- ClassyFire and NPClassifier classifications retrieved via the GNPS API

## Outputs

- Precomputed scoring tables mapping (genome-based class, structure-based class) pairs to relative frequencies
- Machine-readable classification tables usable by NPClassScore in NPLinker (published at github.com/louwenjjr/mibig_classifications)

## How to apply

Download all MIBiG 2.0 entries from mibig.secondarymetabolites.org in json format and extract the SMILES and MIBiG (sub)classes for each entry. Classify the products' SMILES with both ClassyFire (superclass, class, subclass hierarchy) and NPClassifier, retrieving the classifications through the GNPS API. Classify the BGC side using antiSMASH and BiG-SCAPE (sub)classes for each MIBiG BGC. For every (genome-based class, structure-based class) pair, count the co-occurrence of the class terms and convert counts into relative frequencies — the mibig_classifications notebooks count interactions between the two classification types to assess how well each pair matches. Emit the resulting tables as a machine-readable resource consumable by the NPClassScore step in NPLinker, which then assigns each candidate link the maximum relative frequency found for its GCF/spectrum class pair and filters links below the 0.25 cut-off.

## Related tools

- **MIBiG 2.0** (Source of experimentally validated BGCs and their products used to bridge genome-based and structure-based classification ontologies)
- **ClassyFire** (Hierarchical structure-based chemical ontology (superclass, class, subclass) assigned to MIBiG product SMILES)
- **NPClassifier** (Natural-product-specific structure-based classification of MIBiG product SMILES, taking chemistry and biosynthetic pathways into account)
- **antiSMASH** (Genome-based BGC (sub)class assignments for MIBiG entries feeding the genome side of the scoring tables)
- **BiG-SCAPE** (Groups BGCs and provides genome-based class annotations for the genome side of the tables)
- **GNPS API** (Retrieves ClassyFire and NPClassifier chemical compound classifications for the MIBiG SMILES) — https://github.com/NPLinker/nplinker
- **mibig_classifications** (Notebooks and scripts that retrieve both classification types for MIBiG BGCs and count class-term interactions into relative-frequency scoring tables) — github.com/louwenjjr/mibig_classifications

## Examples

```
conda create -n myenv python=3.7.2 rdkit && conda activate myenv && pip install jupyter; jupyter-notebook  # mibig_classifications notebooks retrieve ClassyFire/NPClassifier classes and build the scoring tables
```

## Evaluation signals

- Every MIBiG 2.0 json entry contributes an SMILES and (sub)classes; counts of class-term interactions should cover all entries whose SMILES were successfully classified by both ClassyFire and NPClassifier.
- Table entries are relative frequencies: each (genome-based class, structure-based class) pair value lies between 0 and 1 and reflects the relative count of that matched pair.
- Downstream sanity check: applying the tables via NPClassScore with a 0.25 cut-off should reduce candidate links by roughly 63% on average relative to co-occurrence-based scoring alone, while retaining ~96% of validated links.
- Tables must be consumable by NPLinker's NPClassScore lookup, which assigns the maximum relative frequency found for a candidate link's GCF/spectrum class pair.

## Limitations

- The tables are derived from MIBiG 2.0, so coverage is bounded by the community-curated entries available; the lack of verified genome–metabolome links remains a bottleneck for validating such methods.
- Scoring tables can be further updated as new MIBiG entries and classifications appear (the article notes the tables 'can be further updated').
- Spectra lacking CANOPUS or MolNetEnhancer predictions (1988 across the test datasets) cannot be scored against the tables, so links without structure-based classes may need to be discarded via the filter_missing_scores toggle.

## Evidence

- [results] MIBiG-derived scoring tables from ClassyFire/NPClassifier vs antiSMASH/BiG-SCAPE classes: [evidence span withheld — anchor status: not_found]
- [discussion] MIBiG 2.0 download and extraction workflow: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] ClassyFire and NPClassifier retrieval via GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] mibig_classifications repo: counting interactions between classification types: [evidence span withheld — anchor status: external_doc]
- [intro] Bridging ontologies through MIBiG: "To bridge the different class ontologies resulting from genome and metabolome mining, we used the MIBiG repository that contains experimentally validated BGCs and their products"
