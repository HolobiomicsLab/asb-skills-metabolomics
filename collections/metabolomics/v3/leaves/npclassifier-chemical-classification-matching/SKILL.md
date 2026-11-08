---
name: npclassifier-chemical-classification-matching
description: Use when filtering predicted/machine-generated NP structures (e.g. a
  vendor or combinatorial source mixed with curated NPs) using NP-likeness score,
  NPclassifier class assignments, or sugar-moiety flags as retention criteria.
when_to_use_negative:
- Molecules that occur in any curated source besides SuperNatural II — the filter
  applies only to SuperNatural II-exclusive entries after provenance-based unification
- General structural QC of molecules from all sources (use the ChEMBL structure curation
  pipeline / size 5–210 heavy atoms / connectivity checks instead)
- When no precomputed NP-likeness scores or NPclassifier annotations exist and recomputation
  from synthetic-molecule reference sets (e.g. ZINC15) is not feasible
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: NPclassifier
  role: Assigns natural-product chemical classes; a molecule is kept if NPclassifier
    classifies it as an NP
- name: RDKit
  role: Parses molecule structures and supports sugar-moiety property computation
    for the filter
  repo: https://github.com/Steinbeck-Lab/coconut
- name: CDK
  role: In-build library used to compute the molecular properties (NP-likeness, sugar
    flags) consumed by this filter
- name: MongoDB
  role: Stores unified NPs and derived properties (npl_score, sugar fragment flags)
    queried during filtering
  repo: https://github.com/mSorok/NaturalProductsOnline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:eda552982c712294dd853598c33fa969060cb164ed4a39c840d9e5bd94c8a3ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Steinbeck-Lab/coconut
  tools:
  - CDK
  - MongoDB
  - NPclassifier
  - RDKit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# npclassifier-chemical-classification-matching

## Summary

Reproduces COCONUT's additional quality-control gate for molecules found only in SuperNatural II: keep a molecule only if it has a strictly positive NP-likeness score, is assigned an NP class by NPclassifier, or contains a sugar moiety; exactly 24,880 molecules failing all three criteria were removed.

## When to use

Use when filtering predicted/machine-generated NP structures (e.g. a vendor or combinatorial source mixed with curated NPs) using NP-likeness score, NPclassifier class assignments, or sugar-moiety flags as retention criteria.

## When NOT to use

- Molecules that occur in any curated source besides SuperNatural II — the filter applies only to SuperNatural II-exclusive entries after provenance-based unification
- General structural QC of molecules from all sources (use the ChEMBL structure curation pipeline / size 5–210 heavy atoms / connectivity checks instead)
- When no precomputed NP-likeness scores or NPclassifier annotations exist and recomputation from synthetic-molecule reference sets (e.g. ZINC15) is not feasible

## Inputs

- Set of molecules present in SuperNatural II but absent from all other COCONUT source collections (post InChI-key-based unification)
- Precomputed NP-likeness scores (npl_score) from the COCONUT MongoDB uniqueNaturalProduct collection
- NPclassifier classification labels per molecule
- Sugar-moiety flags (contains_sugar, contains_ring_sugars, contains_linear_sugars) computed with CDK and RDKit

## Outputs

- Retained-molecule structure file of SuperNatural II-only molecules passing the OR-filter
- Removal report tabulating the rejected molecules with their failing criteria (count must equal 24,880)
- Updated COCONUT collection excluding the 24,880 filtered molecules

## How to apply

First isolate the set of molecules present only in the questionable source (here, SuperNatural II) and absent from all other COCONUT source collections, using the provenance-based unification on stereochemistry-free InChI keys. Retrieve for each molecule the precomputed np_likeness score (stored as npl_score in the uniqueNaturalProduct MongoDB collection), NPclassifier class labels, and sugar-moiety flags (contains_sugar, contains_ring_sugars, contains_linear_sugars) computed with the in-build CDK libraries and RDKit. Apply the OR-filter: retain a molecule only if npl_score > 0 OR NPclassifier assigns an NP class OR any sugar flag is true; mark all molecules failing all three criteria for removal. Write the retained molecules to a structure file and tabulate rejected molecules with their failing criteria in a removal report. Validate that the number of removed molecules equals exactly 24,880 as reported in the paper.

## Related tools

- **NPclassifier** (Assigns natural-product chemical classes; a molecule is kept if NPclassifier classifies it as an NP)
- **RDKit** (Parses molecule structures and supports sugar-moiety property computation for the filter) — https://github.com/Steinbeck-Lab/coconut
- **CDK** (In-build library used to compute the molecular properties (NP-likeness, sugar flags) consumed by this filter)
- **MongoDB** (Stores unified NPs and derived properties (npl_score, sugar fragment flags) queried during filtering) — https://github.com/mSorok/NaturalProductsOnline

## Examples

```
db.uniqueNaturalProduct.find({source:'SuperNatural II', npl_score:{$lte:0}, chemicalClass:null, contains_sugar:false})
```

## Evaluation signals

- Number of removed molecules equals exactly 24,880, as reported in the paper
- Every retained SuperNatural II-only molecule satisfies at least one criterion: npl_score > 0, non-empty NPclassifier NP class, or a true sugar-moiety flag
- Every rejected molecule fails all three criteria, recorded in the removal report
- Filtered set is disjoint from molecules present in any other COCONUT source (verified via stereochemistry-free InChI keys)

## Limitations

- The OR-filter depends on precomputed annotations; the COCONUT compiler README notes NP-likeness score recalculation from scratch requires adding a synthetic-molecule dataset (ZINC15 suggested)
- Filter scope is restricted to molecules from a single source (SuperNatural II); applying it globally would remove legitimately curated NPs from other collections
- Structures that fail to parse by RDKit are eliminated from further processing, so annotation-dependent filtering is only as complete as the parseable subset

## Evidence

- [intro] To be kept in COCONUT and be considered as a genuine or predicted NP, such a molecule has to have a strictly positive NP-likeness score, be classified as a NP by NPclassifier: "To be kept in COCONUT and be considered as a genuine or predicted NP, such a molecule has to have a strictly positive NP-likeness score, be classified as a NP by NPclassifier [24]"
- [intro] The 24,880 molecules from SuperNatural II that didn't pass this additional quality control have been removed from COCONUT: [evidence span withheld — anchor status: not_found]
- [intro] Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry.: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [intro] NPs in COCONUT have been analysed for sugar moieties presence, and a deglycosylated structure representation was made available in the database.: "NPs in COCONUT have been analysed for sugar moieties presence, and a deglycosylated structure representation was made available in the database."
- [readme] db.uniqueNaturalProduct.createIndex( {npl_score:1}): [evidence span withheld — anchor status: external_doc]
