---
name: metabolite-molecular-property-retrieval
description: Use when a metabolite namespace (e.g. MNXref) must be annotated with
  molecular properties for permeability classification, and identifiers need translating
  to PubChem CIDs before applying rule-based thresholds.
when_to_use_negative:
- When the task targets a single organism-specific model with curated, experimentally
  verified transport reactions rather than an MNXref-universal gap-filling database.
- When metabolite structures/InChI keys are unavailable, since only ~26% of InChI
  keys translate to PubChem CIDs and untranslatable entries default to 'not likely
  permeable', dominating the classification.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: COMMIT
  role: Provides the permeability classification and the addNamesPermeabilityToGfDb
    function that writes the per-metabolite table and builds the reduced-cost transport-reaction
    subset
  repo: https://github.com/pwendering/COMMIT
- name: PubChem
  role: External database from which the six molecular properties (MW, TPSA, HBD,
    HBA, RB, XLogP3) are retrieved for MNXref metabolites
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:bb7d2b310d612a483cb2700e5417f561bf3169f3754d77837ce280fbf4896bac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COMMIT
  - PubChem
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-molecular-property-retrieval

## Summary

Retrieve six PubChem molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) for all MNXref metabolites and classify each as likely/not likely permeable under Lipinski-style thresholds, producing the per-metabolite permeability table used by COMMIT to assign reduced costs to transport reactions during gap filling.

## When to use

Use when a metabolite namespace (e.g. MNXref) must be annotated with molecular properties for permeability classification, and identifiers need translating to PubChem CIDs before applying rule-based thresholds.

## When NOT to use

- When the task targets a single organism-specific model with curated, experimentally verified transport reactions rather than an MNXref-universal gap-filling database.
- When metabolite structures/InChI keys are unavailable, since only ~26% of InChI keys translate to PubChem CIDs and untranslatable entries default to 'not likely permeable', dominating the classification.

## Inputs

- MNXref metabolite identifier database
- MNXref translation tables chem_prop.tsv and chem_xref.tsv
- PubChem molecular property records (CID-keyed): MW, TPSA, HBD, HBA, RB, XLogP3
- Balanced gap-filling database (data/gap-filling/database/Universal-model-MNXref-balanced.mat)

## Outputs

- Per-metabolite permeability classification table (likely / not likely permeable) for all MNXref metabolites
- Reduced-cost (permeable) transport-reaction subset for the gap-filling database (2534 of 4520 transport reactions marked as candidates in the reference run)

## How to apply

Map MNXref metabolite identifiers to PubChem entries using the MNXref translation tables chem_prop.tsv and chem_xref.tsv, noting that only ~26% of InChI keys translate to PubChem CIDs, yielding molecular properties for 110266 compounds. Retrieve MW, TPSA, HBD, HBA, rotatable bonds (RB) and predicted octanol/water partition coefficient (XLogP3) from PubChem for every MNXref metabolite. Impute missing XLogP3 values with kNN based on the other retrieved properties, since only 11894 entries (11%) carry an associated XLogP3 value. Classify each metabolite as likely permeable with the applied rules HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6, and mark metabolites with no available properties as not likely permeable. Write the per-metabolite permeability table and feed it into the reduced-cost (permeable) transport-reaction subset via the COMMIT function addNamesPermeabilityToGfDb applied to the balanced gap-filling database.

## Related tools

- **COMMIT** (Provides the permeability classification and the addNamesPermeabilityToGfDb function that writes the per-metabolite table and builds the reduced-cost transport-reaction subset) — https://github.com/pwendering/COMMIT
- **PubChem** (External database from which the six molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) are retrieved for MNXref metabolites)

## Examples

```
addNamesPermeabilityToGfDb(dbModel_MNXref_balanced, mnxref_dir)
```

## Evaluation signals

- Every MNXref metabolite in the output table carries a permeability classification consistent with the six thresholds (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6).
- Metabolites with no available properties are marked as not likely permeable.
- Coverage check: molecular properties obtainable for 110266 of the MNXref compounds (~26% InChI-key-to-CID translation rate).
- XLogP3 missing-value check: only 11894 entries (11%) carry an associated XLogP3 value; the remainder must be kNN-imputed before thresholding.
- Transport-reaction subset size sanity check: 2534 of 4520 transport reactions in the gap-filling database are marked as reduced-cost candidates.

## Limitations

- Only ~26% of InChI keys could be translated to PubChem CIDs, leaving many metabolites without properties and thus defaulting to 'not likely permeable'.
- Only 11% of PubChem entries had an associated XLogP3 value, so most XLogP3 inputs to the threshold rules are kNN-imputed rather than measured.
- The Lipinski-style thresholds were applied ad hoc for this study; metabolites lacking any properties are treated as non-permeable, which is a conservative but potentially wrong default.

## Evidence

- [discussion] Lipinski-style thresholds: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] missing properties default: "If no properties were available, the respective metabolite was classified as not likely to be permeable."
- [discussion] property retrieval from PubChem: "rotatable bonds (RB) and predicted octanol/water partition coefficient (XLogP3), were obtained from PubChem [51] for all metabolites in the MNXref database"
- [discussion] coverage and XLogP3 sparsity: "so molecular properties for 110266 compounds could be obtained. Since not all entries had an associated XLogP3 value (only 11894: 11%)"
- [results] reduced-cost candidate count: [evidence span withheld — anchor status: not_found]
