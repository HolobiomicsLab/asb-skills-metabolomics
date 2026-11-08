---
name: smiles-structure-classification-retrieval
description: Use when you have SMILES strings for known metabolites (e.g., extracted
  from MIBiG JSON entries) and need hierarchical NPClassifier/ClassyFire class terms
  to construct or update class-matching frequency tables for BGC–MS/MS linking.
when_to_use_negative:
- The input is raw MS/MS spectra without SMILES structures — in that case CANOPUS
  (run via Sirius v4.9.3 on the GNPS mgf file) or MolNetEnhancer (run externally on
  the GNPS platform) are used to predict structure-based classes instead
- Candidate compounds already carry structure-based class annotations, e.g. when linking
  spectra whose CANOPUS or MolNetEnhancer predictions are already available
- The goal is genome-side BGC class assignment (antiSMASH predictions) rather than
  structure-side chemical classification of metabolites
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: GNPS
  role: API through which the ClassyFire and NPClassifier chemical compound classifications
    are retrieved for each SMILES
- name: ClassyFire
  role: Hierarchical structure-based ontology (superclass, class, subclass plus seven
    more detailed levels) queried for each MIBiG SMILES
- name: NPClassifier
  role: Natural-product-specific structure classifier (taking chemistry and biosynthetic
    pathways into account) queried for each MIBiG SMILES
- name: mibig_classifications
  role: Notebooks and scripts that retrieve the chemical classes, connect BGC and
    structure classifications, and write the relative-count class-matching tables
    used by NPClassScore
  repo: github.com/louwenjjr/mibig_classifications
- name: NPLinker
  role: Platform in which the resulting NPClassScore scoring tables are consumed for
    class-based filtering of BGC–MS/MS links
  repo: github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:a29a5d2fab7e2b033e44a04dc90a136ae0abee7716006885ae5272438b4183fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - GNPS
  - NPClassifier
  - NPLinker
  - antiSMASH
  - mibig_classifications
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# smiles-structure-classification-retrieval

## Summary

Retrieves structure-based chemical class annotations (ClassyFire and NPClassifier superclass/class/subclass) for each compound's SMILES string via the GNPS API, as used to build the MIBiG 2.0 class-matching scoring tables underlying NPClassScore.

## When to use

Use when you have SMILES strings for known metabolites (e.g., extracted from MIBiG JSON entries) and need hierarchical NPClassifier/ClassyFire class terms to construct or update class-matching frequency tables for BGC–MS/MS linking.

## When NOT to use

- The input is raw MS/MS spectra without SMILES structures — in that case CANOPUS (run via Sirius v4.9.3 on the GNPS mgf file) or MolNetEnhancer (run externally on the GNPS platform) are used to predict structure-based classes instead
- Candidate compounds already carry structure-based class annotations, e.g. when linking spectra whose CANOPUS or MolNetEnhancer predictions are already available
- The goal is genome-side BGC class assignment (antiSMASH predictions) rather than structure-side chemical classification of metabolites

## Inputs

- MIBiG 2.0 entries downloaded as JSON files from mibig.secondarymetabolites.org
- SMILES strings extracted from MIBiG entries (parsed with Python json/pandas)
- MIBiG superclass/class/subclass annotations per entry

## Outputs

- NPClassifier and ClassyFire class annotations (superclass/class/subclass) for each SMILES
- 54 class-pairing relative-frequency tables (Table S1 equivalents), of which 28 are the tables consumed by NPClassScore in NPLinker

## How to apply

Download all MIBiG 2.0 entries as JSON files from mibig.secondarymetabolites.org and parse each entry with Python (json/pandas) to extract the SMILES string together with the MIBiG superclass/class/subclass annotations. For each SMILES, retrieve both the ClassyFire and NPClassifier chemical compound classifications through the GNPS API (the mibig_classifications notebooks/scripts wrap this retrieval and use rdkit in a Python 3.7.2 conda environment). Count occurrences of each genome-side BGC class paired with each structure-side chemical class and convert counts to relative frequencies, normalising by the total occurrence of either the genome-based or structure-based class. Write the 54 class-pairing relative-frequency tables to disk, making the 28 tables required by NPClassScore identifiable; these reproduce Table S1 and the class pairings of Fig. 1b–c. Because the tables are empirical frequencies derived from validated BGC–structure pairs, they can be regenerated later to update NPClassScore scoring.

## Related tools

- **GNPS** (API through which the ClassyFire and NPClassifier chemical compound classifications are retrieved for each SMILES)
- **ClassyFire** (Hierarchical structure-based ontology (superclass, class, subclass plus seven more detailed levels) queried for each MIBiG SMILES)
- **NPClassifier** (Natural-product-specific structure classifier (taking chemistry and biosynthetic pathways into account) queried for each MIBiG SMILES)
- **mibig_classifications** (Notebooks and scripts that retrieve the chemical classes, connect BGC and structure classifications, and write the relative-count class-matching tables used by NPClassScore) — github.com/louwenjjr/mibig_classifications
- **NPLinker** (Platform in which the resulting NPClassScore scoring tables are consumed for class-based filtering of BGC–MS/MS links) — github.com/NPLinker/nplinker

## Evaluation signals

- Each of the 54 output tables covers the complete set of genome- and structure-side classes observed across the 1926 MIBiG 2.0 entries
- The subset of 28 tables used by NPClassScore matches the class pairings described in the methods (Fig. 1b–c relationships)
- Every MIBiG 2.0 SMILES has both an NPClassifier and a ClassyFire annotation retrieved via the GNPS API (missing annotations should be accounted for explicitly)
- Table entries are relative frequencies normalised by total occurrence of the genome-based or structure-based class, not raw counts
- Reproduced tables are consistent with the published Table S1 and Fig. 1c class-matching relationships

## Limitations

- Retrieval depends on external classification services (GNPS API access to ClassyFire/NPClassifier); entries without retrievable structure-based classes cannot be scored
- Scoring tables are empirical and derived from the ~1926 MIBiG 2.0 validated pairs; class pairs absent from MIBiG receive no frequency support
- The NPClassScore scoring tables 'can be further updated' (discussion), so tables regenerated from a newer MIBiG release may differ from published Table S1

## Evidence

- [discussion] All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] Series of jupyter notebooks and scripts to retrieve chemical classes (ClassyFire/NPClassifier), as well as BGC classifications for MIBiG BGCs and their products.: [evidence span withheld — anchor status: external_doc]
- [results] The class-matching scoring tables were built by counting class matches across MIBiG entries and normalising by the total occurrence of either the genome-based or the structure-based class: [evidence span withheld — anchor status: not_found]
