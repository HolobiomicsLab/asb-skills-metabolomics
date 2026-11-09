---
name: natural-product-likeness-score-filtering
description: Use when curating a natural products collection that includes predicted
  NPs from SuperNatural II and you must decide which SuperNatural II-only molecules
  qualify as genuine or plausible predicted NPs before adding them to the database.
when_to_use_negative:
- The molecule already originates from a curated source (e.g. ChEBI NPs, AfroDB, BitterDB)
  — this additional QC applies only to molecules found exclusively in SuperNatural
  II
- You are running the general COCONUT structure QC (heavy-atom size 5–210, connectivity,
  pseudo-atoms, hydrogens, valences) rather than deciding NP-likeness of SuperNatural
  II-only entries
- The goal is removing polyfluorinated compounds (64 total) or structures unparseable
  by the ChEMBL curation pipeline (113 total) — those are separate filter steps
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3336
tools:
- name: COCONUT compiler
  role: Reads molecules, checks errors/connectivity, and computes molecular parameters
    including the NP-likeness score stored as npl_score in MongoDB
  repo: https://github.com/mSorok/COCONUT
- name: RDKit
  role: Used with in-build CDK libraries to compute the COCONUT molecular properties
    (np_likeness, sugar-moiety flags) consumed by this filter
- name: NPclassifier
  role: Provides the NP classification labels used as the second criterion of the
    OR-filter
- name: CDK
  role: In-build CDK libraries compute the range of molecular properties and descriptors
    used in the quality-control criteria
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:fd5972dca710ca0115300c9207c25ef2909599c6de379c36d475ea9b23e0e7f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/COCONUT
  tools:
  - CDK
  - COCONUT compiler
  - NPclassifier
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# natural-product-likeness-score-filtering

## Summary

An OR-based quality-control filter for predicted natural products sourced only from SuperNatural II: molecules must have a strictly positive NP-likeness score, OR an NPclassifier NP classification, OR a sugar moiety to be retained in COCONUT; all others are removed.

## When to use

Use when curating a natural products collection that includes predicted NPs from SuperNatural II and you must decide which SuperNatural II-only molecules qualify as genuine or plausible predicted NPs before adding them to the database.

## When NOT to use

- The molecule already originates from a curated source (e.g. ChEBI NPs, AfroDB, BitterDB) — this additional QC applies only to molecules found exclusively in SuperNatural II
- You are running the general COCONUT structure QC (heavy-atom size 5–210, connectivity, pseudo-atoms, hydrogens, valences) rather than deciding NP-likeness of SuperNatural II-only entries
- The goal is removing polyfluorinated compounds (64 total) or structures unparseable by the ChEMBL curation pipeline (113 total) — those are separate filter steps

## Inputs

- Set of molecules present only in SuperNatural II after provenance-based unification of COCONUT sources
- Precomputed NP-likeness scores (npl_score field in the uniqueNaturalProduct MongoDB collection)
- NPclassifier NP class assignments per molecule
- Sugar-moiety flags (contains_sugar, contains_ring_sugars, contains_linear_sugars) from COCONUT's sugar analysis

## Outputs

- Filtered set of SuperNatural II-only molecules retained as genuine or predicted NPs
- Removal report of the 24,880 rejected molecules with their failing criteria
- Updated structure file of retained molecules written back to the collection

## How to apply

First isolate the set of molecules present in SuperNatural II but absent from all other COCONUT source collections, using the provenance-based unification of InChI keys without stereochemistry. For each molecule, retrieve the precomputed np_likeness score (npl_score, computed against a synthetic-molecule reference set such as ZINC15), the NPclassifier NP class labels, and the sugar-moiety flags produced during COCONUT's sugar analysis. Apply the OR-rule: retain a molecule only if the NP-likeness score is strictly positive (>0), OR NPclassifier assigns it an NP class, OR it contains a sugar moiety (contains_sugar / contains_ring_sugars / contains_linear_sugars). Mark all molecules failing all three criteria for removal and write them to a removal report; in the COCONUT curation this removed exactly 24,880 SuperNatural II-only molecules, which serves as the validation target when reproducing the pipeline.

## Related tools

- **COCONUT compiler** (Reads molecules, checks errors/connectivity, and computes molecular parameters including the NP-likeness score stored as npl_score in MongoDB) — https://github.com/mSorok/COCONUT
- **RDKit** (Used with in-build CDK libraries to compute the COCONUT molecular properties (np_likeness, sugar-moiety flags) consumed by this filter)
- **NPclassifier** (Provides the NP classification labels used as the second criterion of the OR-filter)
- **CDK** (In-build CDK libraries compute the range of molecular properties and descriptors used in the quality-control criteria)

## Examples

```
java -Xmx16288m -jar coconut-0.0.1-SNAPSHOT.jar data sm/sm.tsv fragments/fragment_without_sugar.txt fragments/fragment_with_sugar.txt importCOCONUTids coconut_ids_june2020.csv > logs.txt &
```

## Evaluation signals

- The count of removed SuperNatural II-only molecules equals exactly 24,880, as reported in the paper
- Every retained molecule satisfies at least one criterion: npl_score > 0, a non-empty NPclassifier NP class, or a true sugar-moiety flag
- Every removed molecule fails all three criteria, and its failing criteria are tabulated in the removal report
- The filtered set is a strict subset of the SuperNatural II-only set established by InChI-key (no stereochemistry) unification
- Retained molecules round-trip into the uniqueNaturalProduct MongoDB collection with valid coconut_id and inchikey

## Limitations

- The filter is applied only to molecules occurring in SuperNatural II and no other COCONUT source, so it cannot be benchmarked against multi-source molecules
- NP-likeness score recomputation requires adding a synthetic-molecule dataset (e.g. ZINC15) — 'not adding any SM will not affect COCONUT, unless there is a need of re-calculating the NP-likeness score from scratch'
- The OR-logic is generous: a molecule passing only one weak criterion (e.g. marginally positive NP-likeness) is retained

## Evidence

- [intro] Additional QC filter definition: "To be kept in COCONUT and be considered as a genuine or predicted NP, such a molecule has to have a strictly positive NP-likeness score, be classified as a NP by NPclassifier [24]"
- [intro] Removal count validation: [evidence span withheld — anchor status: not_found]
- [intro] Scope of the filter: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [readme] NP-likeness score storage: [evidence span withheld — anchor status: external_doc]
- [readme] Synthetic-molecule requirement for NP-likeness: [evidence span withheld — anchor status: not_found]
- [intro] Sugar-moiety criterion availability: "NPs in COCONUT have been analysed for sugar moieties presence, and a deglycosylated structure representation was made available in the database."
