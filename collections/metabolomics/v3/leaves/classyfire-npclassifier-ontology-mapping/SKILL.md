---
name: classyfire-npclassifier-ontology-mapping
description: Use when MS/MS spectra from GNPS molecular networking and BGC products
  need class annotations in ClassyFire and NPClassifier ontologies to match classes
  across genome- and metabolome-based mining, e.g. to build NPClassScore scoring tables
  or filter BGC–MS/MS links.
when_to_use_negative:
- Spectra with masses above 850 Da — CANOPUS is only applied below the --maxmz 850
  cut-off because fragmentation-tree calculation is time-consuming and larger metabolites
  are not well handled; mass-spectral embeddings such as Spec2Vec or MS2DeepScore
  are suggested instead
- When classes can be derived from the molecular network itself — MolNetEnhancer has
  to be run externally (e.g. on the GNPS platform) and cannot currently be run within
  NPLinker
- When the goal is BGC classification rather than chemical class prediction of products/spectra
  — the mibig_classifications workflow is designed to connect both classification
  types, not replace BGC classifiers
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: CANOPUS
  role: Predicts structure-based ClassyFire and NPClassifier classes directly from
    MS/MS spectra via SIRIUS v4.9.3, restricted to spectra below 850 Da
  repo: github.com/kaibioinfo/canopus_treemap
- name: NPLinker
  role: Platform hosting the CANOPUS run on GNPS mgf files and the NPClassScore method
    that consumes the matched class ontologies
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Produces the spectra.mgf from molecular networking and provides the API through
    which ClassyFire and NPClassifier classifications of MIBiG product structures
    are retrieved
- name: ClassyFire
  role: Hierarchical ontology (superclass, class, subclass, plus seven more detailed
    levels) used as the chemical class target for both BGC products and spectra
- name: NPClassifier
  role: Natural-product-specific ontology taking chemistry and biosynthetic pathways
    into account, used alongside ClassyFire for class matching
- name: mibig_classifications
  role: Notebooks and scripts to retrieve ClassyFire/NPClassifier and BGC classifications
    for MIBiG BGCs and count matched class-pair interactions for the scoring tables
  repo: github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:6a92a586298b7de91fc36ec5d88942cd38228604c2559330bda8c4114288af9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ClassyFire
  - GNPS
  - NPClassifier
  - NPLinker
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# classyfire-npclassifier-ontology-mapping

## Summary

Assigns natural product chemical classes to BGC products and MS/MS spectra in the ClassyFire and NPClassifier ontologies so the two classification schemes can be matched for BGC–metabolite linking (e.g. NPClassScore). Structure-based classes for spectra are predicted with CANOPUS via SIRIUS within NPLinker.

## When to use

Use when MS/MS spectra from GNPS molecular networking and BGC products need class annotations in ClassyFire and NPClassifier ontologies to match classes across genome- and metabolome-based mining, e.g. to build NPClassScore scoring tables or filter BGC–MS/MS links.

## When NOT to use

- Spectra with masses above 850 Da — CANOPUS is only applied below the --maxmz 850 cut-off because fragmentation-tree calculation is time-consuming and larger metabolites are not well handled; mass-spectral embeddings such as Spec2Vec or MS2DeepScore are suggested instead
- When classes can be derived from the molecular network itself — MolNetEnhancer has to be run externally (e.g. on the GNPS platform) and cannot currently be run within NPLinker
- When the goal is BGC classification rather than chemical class prediction of products/spectra — the mibig_classifications workflow is designed to connect both classification types, not replace BGC classifiers

## Inputs

- spectra.mgf file produced by GNPS molecular networking (from the gnps directory of the NPLinker working directory)
- MIBiG 2.0 entries in JSON format from mibig.secondarymetabolites.org (SMILES and MIBiG (sub)classes)
- molecular_families.tsv mapping spectra to molecular families (MFs)

## Outputs

- class_annotations.tsv with per-spectrum and per-MF ClassyFire and NPClassifier predictions at super-, class-, and subclass-levels
- Class–class interaction count tables linking BGC classes to chemical classes (basis of the NPClassScore scoring tables)

## How to apply

Bridge the two ontologies using MIBiG 2.0 entries downloaded from mibig.secondarymetabolites.org in JSON format, extracting SMILES and MIBiG (sub)classes, then retrieving ClassyFire and NPClassifier classifications of the product structures through the GNPS API. For spectra, run CANOPUS through SIRIUS v4.9.3 directly on the spectra.mgf produced by GNPS molecular networking (loaded from the gnps directory of the NPLinker working directory) with the 'formula zodiac structure canopus' setting and --maxmz 850, since fragmentation-tree computation is time-consuming and limits coverage to compounds below 850 Da. Extract per-spectrum ClassyFire and NPClassifier predictions at super-, class-, and subclass-levels, keeping classes at a probability cut-off of 0.5 for ClassyFire and 0.33 for NPClassifier. Aggregate per-spectrum predictions into molecular-family-level annotations using molecular_families.tsv, retaining a class at the MF level only when it occurs in at least 20% of the spectra in that family. Write both per-spectrum and per-MF predictions to class_annotations.tsv; matched class-term pairs and their relative co-occurrence counts then underpin NPClassScore scoring tables used to rule out unreliable BGC–compound matches.

## Related tools

- **CANOPUS** (Predicts structure-based ClassyFire and NPClassifier classes directly from MS/MS spectra via SIRIUS v4.9.3, restricted to spectra below 850 Da) — github.com/kaibioinfo/canopus_treemap
- **NPLinker** (Platform hosting the CANOPUS run on GNPS mgf files and the NPClassScore method that consumes the matched class ontologies) — github.com/NPLinker/nplinker
- **GNPS** (Produces the spectra.mgf from molecular networking and provides the API through which ClassyFire and NPClassifier classifications of MIBiG product structures are retrieved)
- **ClassyFire** (Hierarchical ontology (superclass, class, subclass, plus seven more detailed levels) used as the chemical class target for both BGC products and spectra)
- **NPClassifier** (Natural-product-specific ontology taking chemistry and biosynthetic pathways into account, used alongside ClassyFire for class matching)
- **mibig_classifications** (Notebooks and scripts to retrieve ClassyFire/NPClassifier and BGC classifications for MIBiG BGCs and count matched class-pair interactions for the scoring tables) — github.com/louwenjjr/mibig_classifications

## Examples

```
sirius "formula zodiac structure canopus" --input spectra.mgf --maxmz 850
```

## Evaluation signals

- class_annotations.tsv exists, parses as a well-formed TSV, and contains the expected class-level columns with both per-spectrum and per-MF rows
- Every MF-level entry contains only classes that occur in at least 20% of the spectra of that molecular family in molecular_families.tsv
- Per-spectrum predictions respect the probability cut-offs: 0.5 for ClassyFire and 0.33 for NPClassifier classes
- No spectra with m/z above 850 are present in the CANOPUS-derived annotations (consistent with the --maxmz 850 setting)
- The count of spectra without structure-based class predictions is tracked so downstream NPClassScore filtering can consistently discard links without classes

## Limitations

- CANOPUS coverage is limited by the 850 Da mass cut-off; 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction, and not filtering out links without classes decreases average candidate links per GCF by 49%
- Fragmentation-tree calculation makes CANOPUS time-consuming, motivating the lower-mass restriction
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally, for example on the GNPS platform

## Evidence

- [discussion] CANOPUS run on GNPS mgf: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [discussion] GNPS API classifications: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [intro] ClassyFire ontology levels: "One of these is ClassyFire: a hierarchical ontology consisting of superclass, class, and subclass categories and seven more detailed levels"
- [intro] NPClassifier purpose: "Recently, NPClassifier was developed specifically for NP classifications, taking both chemistry and biosynthetic pathways into account"
- [intro] Bridging ontologies via MIBiG: "To bridge the different class ontologies resulting from genome and metabolome mining, we used the MIBiG repository that contains experimentally validated BGCs and their products"
- [readme] NPC prediction support in CANOPUS: [evidence span withheld — anchor status: external_doc]
- [readme] Purpose of matched classes: [evidence span withheld — anchor status: external_doc]
