---
name: small-molecule-structure-database-preparation
description: Use when tandem mass spectra (e.g., from GNPS/MassIVE datasets like MSV000079450)
  must be searched against known molecular structure databases and no prepared structure-DB
  target set is yet available.
when_to_use_negative:
- The analysis goal is spectral library search against reference spectra rather than
  in silico search against molecular structure databases
- Spectra are so large/heavy (>1000 Da) or small (<400 Da) that ML-based search methods
  are known to perform poorly, per the article's scope notes
- Only a small set of molecules with reference spectra is needed — spectral libraries
  may suffice since current libraries cover tens of thousands of molecules
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Downstream consumer of the prepared structure database; scores molecule-spectrum
    pairs with a learned probabilistic fragmentation model
  repo: https://github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:3c5ecf75c4bddf58b525f08649732c9b6a64c98602c2311bd51020530b0a2a4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mohimanilab/molDiscovery
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# small-molecule-structure-database-preparation

## Summary

Prepare a molecular structure database (e.g., AllDB of 719,958 small-molecule structures drawn from sources like PubChem and the Dictionary of Natural Products) as the target reference for mass spectral database search with molDiscovery against large tandem MS datasets such as the ~8 million GNPS spectra.

## When to use

Use when tandem mass spectra (e.g., from GNPS/MassIVE datasets like MSV000079450) must be searched against known molecular structure databases and no prepared structure-DB target set is yet available.

## When NOT to use

- The analysis goal is spectral library search against reference spectra rather than in silico search against molecular structure databases
- Spectra are so large/heavy (>1000 Da) or small (<400 Da) that ML-based search methods are known to perform poorly, per the article's scope notes
- Only a small set of molecules with reference spectra is needed — spectral libraries may suffice since current libraries cover tens of thousands of molecules

## Inputs

- Molecular structure database entries (e.g., AllDB with 719,958 small-molecule structures)
- Molecule records from PubChem or the Dictionary of Natural Products (DNP) as candidate sources

## Outputs

- Prepared small-molecule structure database ready for molDiscovery spectral search
- Target/decoy structure pairs supporting 0% FDR filtering of identifications

## How to apply

Assemble or obtain the molecular structure database to be used as the search target — in the molDiscovery GNPS benchmark this is AllDB, containing 719,958 small-molecule structures, with candidate molecules also available from sources such as PubChem and the Dictionary of Natural Products (DNP). Normalize the structures so each entry can be scored against a tandem mass spectrum as a molecule-spectrum pair. The prepared database is then consumed by molDiscovery's learned probabilistic fragmentation model (or a baseline like Dereplicator+) which scores every molecule-spectrum pair. Identifications are subsequently filtered at 0% false discovery rate (FDR) before counting matched spectra and unique compounds, so the database must be complete and deduplicated enough for reliable target-decoy FDR estimation. Validate the preparation by confirming the structure count (e.g., 719,958 for AllDB) and that downstream search yields the expected identifications (3,185 unique molecules at 0% FDR vs. 542 for Dereplicator+).

## Related tools

- **molDiscovery** (Downstream consumer of the prepared structure database; scores molecule-spectrum pairs with a learned probabilistic fragmentation model) — https://github.com/mohimanilab/molDiscovery

## Evaluation signals

- Database structure count matches the expected 719,958 small-molecule structures for AllDB
- Downstream molDiscovery search of the ~8 million GNPS spectra annotates ~56,971 spectra at 0% FDR (8x over Dereplicator+'s 7,684)
- Unique compound count at 0% FDR reaches 3,185, an approximately six-fold increase over Dereplicator+'s 542
- Identifications are filtered at 0% false discovery rate before any counting, confirming FDR machinery was applied to the database search
- Reproduced counts match the reported figures recorded in moldiscovery_gnps_search_results.csv

## Limitations

- molDiscovery currently covers only a limited number of bond types and rearrangement rules, so some structures in the database may be poorly scored
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Default fragmentation model was trained on combined CID/HCD positive-mode spectra; instrument type, dissociation technique, and collision energy are not fully modeled
- Lower annotation accuracy in the low mass range (<600 Da) partly because fewer fragments are generated for small molecules

## Evidence

- [intro] 8 million tandem mass spectra from the Global Natural Product Social molecular networking infrastructure (GNPS): "8 million tandem mass spectra from the Global Natural Product Social molecular networking infrastructure (GNPS)"
- [intro] molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [intro] This approach enables small molecule identification in known databases, such as PubChem: "This approach enables small molecule identification in known databases, such as PubChem"
- [readme] To test molDiscovery for academic users, please visit: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
