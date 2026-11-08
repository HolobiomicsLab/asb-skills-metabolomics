---
name: molecule-provenance-based-subset-selection
description: Use when a merged multi-source chemical collection contains a source
  whose entries need an extra QC gate, and the subset must first be isolated by provenance
  (present only in that one source) before applying per-molecule property filters.
when_to_use_negative:
- Applying the filter to molecules present in multiple COCONUT sources — the additional
  QC gate is defined only for molecules occurring solely in SuperNatural II
- As a substitute for the general structure QC pipeline (size 5–210 heavy atoms, connectivity,
  pseudo-atoms, hydrogens, bonds, valences) that applies to all molecules regardless
  of provenance
- When the goal is deduplication or merging of entries across sources rather than
  provenance-based subset selection and retention decisions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: COCONUT compiler
  role: Java/JAR pipeline that computes molecular parameters and properties (np_likeness,
    sugar-moiety flags) and stores molecules with provenance in the Mongo database
  repo: https://github.com/mSorok/COCONUT
- name: MongoDB
  role: Stores original data, unified NPs and derived annotations; queried to select
    molecules by source and simpleInchiKey
- name: RDKit
  role: Used for parsing and computing molecular properties (npl_score, sugar flags)
    used by the OR-filter
- name: NPclassifier
  role: Assigns NP classification labels used as one of the three retention criteria
- name: CDK
  role: In-build library used to compute the molecular properties and annotations
    consumed by the filter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:5dbb52241d8d05357a859ac8fc413afbfbadd7b05451804f94c1706363039c4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/COCONUT
  tools:
  - CDK
  - COCONUT compiler
  - MongoDB
  - NPclassifier
  - RDKit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# molecule-provenance-based-subset-selection

## Summary

Selects a molecule subset defined by collection provenance — e.g., molecules present in SuperNatural II but absent from all other COCONUT source collections — using the provenance-based unification of molecules by InChI keys without stereochemistry, then applies a source-specific quality gate (positive NP-likeness score OR NPclassifier NP class OR sugar moiety) to decide retention.

## When to use

Use when a merged multi-source chemical collection contains a source whose entries need an extra QC gate, and the subset must first be isolated by provenance (present only in that one source) before applying per-molecule property filters.

## When NOT to use

- Applying the filter to molecules present in multiple COCONUT sources — the additional QC gate is defined only for molecules occurring solely in SuperNatural II
- As a substitute for the general structure QC pipeline (size 5–210 heavy atoms, connectivity, pseudo-atoms, hydrogens, bonds, valences) that applies to all molecules regardless of provenance
- When the goal is deduplication or merging of entries across sources rather than provenance-based subset selection and retention decisions

## Inputs

- MongoDB COCONUT dump (sourceNaturalProduct and uniqueNaturalProduct collections) with per-molecule source provenance and InChI keys
- Precomputed per-molecule property annotations: np_likeness (npl_score), NPclassifier NP classification labels, sugar-moiety flags (contains_sugar, contains_ring_sugars, contains_linear_sugars)
- Source collection assignment mapping (e.g., SuperNatural II vs. all other COCONUT sources)

## Outputs

- Retained-molecule structure file limited to SuperNatural II-only molecules passing the OR-filter
- Removal report tabulating the rejected molecules with their failing criteria (expected: 24,880 molecules removed from SuperNatural II)

## How to apply

First unify molecules across sources by the identity of their InChI keys without stereochemistry, as COCONUT does, and record for each molecule which source collections it came from. Query the MongoDB sourceNaturalProduct/uniqueNaturalProduct collections (indexed on source and simpleInchiKey) to select the molecules present in SuperNatural II but absent from every other collection. For each molecule in that subset, retrieve the precomputed np_likeness score, NPclassifier NP classification labels, and sugar-moiety flags (contains_sugar, contains_ring_sugars, contains_linear_sugars) computed with the in-build CDK libraries and RDKit. Apply the OR-filter: retain a molecule only if its NP-likeness score is strictly positive, OR NPclassifier assigns it an NP class, OR it contains a sugar moiety; mark all molecules failing all three criteria for removal. Write the retained molecules to a structure file, tabulate the rejected molecules with their failing criteria in a removal report, and verify the removed count (24,880 for COCONUT as reported in the paper) as a validation check.

## Related tools

- **COCONUT compiler** (Java/JAR pipeline that computes molecular parameters and properties (np_likeness, sugar-moiety flags) and stores molecules with provenance in the Mongo database) — https://github.com/mSorok/COCONUT
- **MongoDB** (Stores original data, unified NPs and derived annotations; queried to select molecules by source and simpleInchiKey)
- **RDKit** (Used for parsing and computing molecular properties (npl_score, sugar flags) used by the OR-filter)
- **NPclassifier** (Assigns NP classification labels used as one of the three retention criteria)
- **CDK** (In-build library used to compute the molecular properties and annotations consumed by the filter)

## Examples

```
java -Xmx16288m -jar coconut-0.0.1-SNAPSHOT.jar data sm/sm.tsv fragments/fragment_without_sugar.txt fragments/fragment_with_sugar.txt importCOCONUTids coconut_ids_june2020.csv
```

## Evaluation signals

- The set of selected molecules is exactly the intersection of 'present in SuperNatural II' and 'absent from all other source collections' after InChI-key (no stereochemistry) unification
- Every removed molecule fails all three criteria: non-positive NP-likeness score, no NPclassifier NP class, and no sugar moiety
- The number of removed molecules equals exactly 24,880, matching the count reported in the paper
- The removal report lists, for each rejected molecule, the specific failing criteria
- Retained molecules have at least one of: strictly positive npl_score, an NPclassifier class, or a sugar-moiety flag set to true

## Limitations

- The 24,880 removal count is specific to the COCONUT version/paper; counts will differ for other releases or source compositions
- Depends on correct provenance annotation; molecules mis-unified across sources (e.g., stereochemistry handling in InChI keys) will leak into or out of the subset
- The OR-filter cannot be recomputed from scratch without the synthetic-molecule dataset used for NP-likeness scoring (README notes ZINC15 is suggested for re-calculating NP-likeness score)

## Evidence

- [intro] Provenance-based unification defines the subset: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [intro] The OR-filter criteria: "To be kept in COCONUT and be considered as a genuine or predicted NP, such a molecule has to have a strictly positive NP-likeness score, be classified as a NP by NPclassifier [24]"
- [intro] Expected removal count and population: [evidence span withheld — anchor status: not_found]
- [intro] Molecule unification on InChI keys without stereochemistry: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [intro] Properties are computed with in-build CDK libraries: "a range of molecular properties, descriptors and fingerprints (full list in Table 2) are computed using the in-build CDK [13] libraries"
- [intro] Sugars are analysed as part of the pipeline: "NPs in COCONUT have been analysed for sugar moieties presence, and a deglycosylated structure representation was made available in the database."
- [readme] NP-likeness score is stored and indexed in MongoDB: [evidence span withheld — anchor status: external_doc]
