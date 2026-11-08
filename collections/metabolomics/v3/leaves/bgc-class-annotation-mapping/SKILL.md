---
name: bgc-class-annotation-mapping
description: Use when linking BGCs to MS/MS spectra requires a mapping between genome-side
  BGC classes and structure-side chemical classes, or when reproducing/ updating NPClassScore
  scoring tables from MIBiG entries.
when_to_use_negative:
- When candidate BGC–MS/MS links lack structure-based classes entirely (e.g., 1988
  MS/MS spectra across the datasets had no CANOPUS or MolNetEnhancer prediction) —
  the class-matching filter cannot score them.
- When the goal is spectral similarity linking of large metabolites beyond the ~850
  Da CANOPUS coverage limit — mass-spectral embeddings such as Spec2Vec or MS2DeepScore
  are better suited.
- When you need to score links between class terms not represented in the MIBiG-derived
  tables — the tables only cover genome- and structure-side classes observed across
  MIBiG 2.0.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIBiG 2.0
  role: Source of experimentally validated BGC–product pairs whose class co-occurrences
    are counted
- name: antiSMASH
  role: Genome-side BGC class predictions cross-referenced to MIBiG classes
- name: ClassyFire
  role: Structure-side hierarchical chemical classification retrieved per SMILES via
    the GNPS API
- name: NPClassifier
  role: Structure-side natural-product classification retrieved per SMILES via the
    GNPS API
- name: NPLinker
  role: Platform in which the class-matching tables are consumed as NPClassScore to
    filter BGC–MS/MS links
  repo: https://github.com/NPLinker/nplinker
- name: mibig_classifications
  role: Jupyter notebooks and scripts that retrieve BGC/compound classifications and
    build the class-matching counting tables
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:95dbba0d39a164cfbf211febda7f43d82725c922b7c3caf892719858e65a754f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - ClassyFire
  - MIBiG 2.0
  - NPClassifier
  - NPLinker
  - antiSMASH
  - mibig_classifications
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bgc-class-annotation-mapping

## Summary

Bridges genome-based BGC class ontologies (e.g., antiSMASH/MIBiG classes) and structure-based chemical ontologies (NPClassifier/ClassyFire) by counting class co-occurrences across experimentally validated MIBiG BGC-product pairs and normalising them into relative-frequency class-matching scoring tables used by NPClassScore.

## When to use

Use when linking BGCs to MS/MS spectra requires a mapping between genome-side BGC classes and structure-side chemical classes, or when reproducing/ updating NPClassScore scoring tables from MIBiG entries.

## When NOT to use

- When candidate BGC–MS/MS links lack structure-based classes entirely (e.g., 1988 MS/MS spectra across the datasets had no CANOPUS or MolNetEnhancer prediction) — the class-matching filter cannot score them.
- When the goal is spectral similarity linking of large metabolites beyond the ~850 Da CANOPUS coverage limit — mass-spectral embeddings such as Spec2Vec or MS2DeepScore are better suited.
- When you need to score links between class terms not represented in the MIBiG-derived tables — the tables only cover genome- and structure-side classes observed across MIBiG 2.0.

## Inputs

- MIBiG 2.0 entries downloaded as JSON from mibig.secondarymetabolites.org (1926 entries)
- SMILES strings of MIBiG products
- MIBiG superclass/class/subclass annotations
- antiSMASH-predicted BGC classes for the MIBiG entries

## Outputs

- 54 class-pairing relative-frequency scoring tables (28 required by NPClassScore), reproducing Table S1 and the class-matching relationships of Fig. 1b–c
- MIBiG BGC and compound classification mappings (as maintained in the mibig_classifications repository)

## How to apply

Download all MIBiG 2.0 entries as JSON from mibig.secondarymetabolites.org and parse each entry (json/pandas) to extract the SMILES string and the MIBiG superclass/class/subclass annotations. Map antiSMASH-predicted BGC classes for each entry and cross-reference them to the MIBiG classes on the genome side, then retrieve NPClassifier and ClassyFire classifications for each SMILES through the GNPS API on the structure side. Count occurrences of each genome-side BGC class paired with each structure-side chemical class and convert the counts to relative frequencies by normalising over the total occurrence of either the genome-based or the structure-based class, mirroring Fig. 1c and Table S1. Write the resulting 54 class-pairing relative-frequency tables to disk; the 28 tables consumed by NPClassScore (implemented in NPLinker) must be identifiable. The rationale is that empirically observed class matches in validated MIBiG pairs empirically quantify how well a BGC class matches a compound class, so improbable class pairings can be filtered (e.g., NPClassScore cut-off 0.25) from candidate BGC–MS/MS links.

## Related tools

- **MIBiG 2.0** (Source of experimentally validated BGC–product pairs whose class co-occurrences are counted)
- **antiSMASH** (Genome-side BGC class predictions cross-referenced to MIBiG classes)
- **ClassyFire** (Structure-side hierarchical chemical classification retrieved per SMILES via the GNPS API)
- **NPClassifier** (Structure-side natural-product classification retrieved per SMILES via the GNPS API)
- **NPLinker** (Platform in which the class-matching tables are consumed as NPClassScore to filter BGC–MS/MS links) — https://github.com/NPLinker/nplinker
- **mibig_classifications** (Jupyter notebooks and scripts that retrieve BGC/compound classifications and build the class-matching counting tables) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
pip install git+https://github.com/nplinker/nplinker@dev
```

## Evaluation signals

- Each of the 54 output tables must cover the complete set of genome- and structure-side classes observed across the 1926 MIBiG 2.0 entries.
- The subset of 28 tables used by NPClassScore must match the class pairings described in the methods (Fig. 1b–c / Table S1).
- Table entries must be relative frequencies (class-match counts normalised by total occurrence of either the genome-based or structure-based class), not raw counts.
- Genome-side classes must derive from cross-referenced antiSMASH and MIBiG annotations, not from an arbitrary external ontology.
- Validation links retained in benchmark datasets (e.g., 96% of validated links in three PoDP datasets retained after NPClassScore filtering) indicate the tables behave as published.

## Limitations

- Coverage is bounded by MIBiG 2.0: classes absent from the 1926 validated entries have no empirical match frequencies and cannot be scored.
- The mapping relies on the availability of structure-based classifications; spectra without CANOPUS or MolNetEnhancer predictions must be discarded or handled explicitly (not filtering decreases average candidate links per GCF by 49%).
- The scoring tables can be further updated as MIBiG grows, so results are snapshot-dependent rather than fixed.

## Evidence

- [discussion] All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] These two types of classifications are then connected to eachother by counting their interactions, ideally matching certain chemical and BGC classes.: [evidence span withheld — anchor status: external_doc]
- [readme] Based on the relative counts of each matched pair of class-terms it is assessed how well the pair of class-terms match to eachother.: [evidence span withheld — anchor status: external_doc]
- [intro] To bridge the different class ontologies resulting from genome and metabolome mining, we used the MIBiG repository that contains experimentally validated BGCs and their products: "To bridge the different class ontologies resulting from genome and metabolome mining, we used the MIBiG repository that contains experimentally validated BGCs and their products"
