---
name: knn-imputation-of-missing-xlogp3
description: Use when a per-metabolite permeability or rule-based property classification
  requires XLogP3, but the descriptor is missing for the large majority of PubChem-mapped
  metabolite identifiers and correlated descriptors (MW, TPSA, HBD, HBA, RB) are available.
when_to_use_negative:
- When XLogP3 coverage is already adequate for the metabolite set — imputation only
  becomes necessary because 89% of MNXref entries lack an associated XLogP3 value.
- When metabolite identifiers cannot be translated to PubChem CIDs at all (no feature
  vector for kNN); in that case the article's convention is to classify the metabolite
  as not likely permeable rather than impute.
- When the analysis needs experimentally measured membrane permeability — this pipeline
  produces a Lipinski-style rule proxy, not a biophysical permeability measurement.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: COMMIT
  role: Hosts the permeability pipeline that consumes the imputed XLogP3 table; addNamesPermeabilityToGfDb
    writes the per-metabolite classification into the gap-filling database.
  repo: https://github.com/pwendering/COMMIT
- name: PubChem
  role: Source of the six molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) retrieved
    for all MNXref metabolites.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:e3e95bd3a0f1172f44d2a4208a39fbb40b9e6c7b08c95a673b76281f93020fe9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COMMIT
  - PubChem
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kNN imputation of missing XLogP3

## Summary

Fills missing predicted octanol/water partition coefficients (XLogP3) for MNXref metabolites by k-nearest-neighbour imputation over the other PubChem molecular properties (MW, TPSA, HBD, HBA, RB), because only 11894 of 110266 PubChem-mapped MNXref compounds (11%) carry an XLogP3 value. The completed property table feeds the Lipinski-style permeability classification used by COMMIT to mark transport reactions as candidates for reduced cost.

## When to use

Use when a per-metabolite permeability or rule-based property classification requires XLogP3, but the descriptor is missing for the large majority of PubChem-mapped metabolite identifiers and correlated descriptors (MW, TPSA, HBD, HBA, RB) are available.

## When NOT to use

- When XLogP3 coverage is already adequate for the metabolite set — imputation only becomes necessary because 89% of MNXref entries lack an associated XLogP3 value.
- When metabolite identifiers cannot be translated to PubChem CIDs at all (no feature vector for kNN); in that case the article's convention is to classify the metabolite as not likely permeable rather than impute.
- When the analysis needs experimentally measured membrane permeability — this pipeline produces a Lipinski-style rule proxy, not a biophysical permeability measurement.

## Inputs

- MNXref metabolite identifier set (chem_prop.tsv, chem_xref.tsv translation tables)
- PubChem molecular property records: molecular weight (MW), topological polar surface area (TPSA), hydrogen bond donors (HBD), hydrogen bond acceptors (HBA), rotatable bonds (RB), predicted octanol/water partition coefficient (XLogP3)
- Balanced gap-filling database (data/gap-filling/database/Universal-model-MNXref-balanced.mat)

## Outputs

- Per-metabolite permeability table with imputed XLogP3 values and likely/not-likely-permeable classification
- Reduced-cost (permeable) transport-reaction subset added to Universal-model-MNXref-balanced.mat (2534 of 4520 transport reactions marked as candidates)

## How to apply

First map MNXref metabolite identifiers to PubChem entries via the MNXref translation tables (chem_prop.tsv, chem_xref.tsv); expect only ~26% of InChI keys to yield PubChem CIDs, giving properties for ~110266 compounds. Retrieve MW, TPSA, HBD, HBA, rotatable bonds (RB) and XLogP3 from PubChem; since only 11894 entries (11%) carry XLogP3, impute the missing values with kNN using the other five retrieved properties as features. Feed the completed table into the Lipinski-style permeability rules (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6) to label each metabolite likely or not likely permeable. Metabolites for which no properties at all are available bypass imputation and are classified as not likely permeable. The resulting per-metabolite permeability table defines the reduced-cost (permeable) transport-reaction subset in the gap-filling database via the COMMIT function addNamesPermeabilityToGfDb.

## Related tools

- **COMMIT** (Hosts the permeability pipeline that consumes the imputed XLogP3 table; addNamesPermeabilityToGfDb writes the per-metabolite classification into the gap-filling database.) — https://github.com/pwendering/COMMIT
- **PubChem** (Source of the six molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) retrieved for all MNXref metabolites.)

## Evaluation signals

- Every MNXref metabolite in the output permeability table carries a classification consistent with the six thresholds (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6).
- No missing XLogP3 values remain among metabolites that have at least one PubChem property; metabolites with no properties are marked not likely permeable.
- Sanity check on input coverage: only 11894 entries (11%) should carry native XLogP3 and ~110266 compounds should have any properties.
- Downstream count check: applying the rules marks 2534 of 4520 transport reactions in the gap-filling database as reduced-cost candidates.
- Imputed XLogP3 values should fall in chemically plausible ranges compatible with the −0.4 to 5.6 rule window used downstream.

## Limitations

- Only ~26% of InChI keys translate to PubChem CIDs, so molecular properties exist for just 110266 MNXref compounds; many metabolites remain property-less and are forced to the not-permeable class.
- Imputation quality depends on the five retained descriptors; no validation of imputed XLogP3 accuracy against measured values is documented.
- The downstream Lipinski-style thresholds were chosen as applied rules, and the gap-filling penalty weights more broadly were tuned ad hoc until satisfying results were obtained, without a systematic selection procedure.

## Evidence

- [other] Impute missing XLogP3 values with kNN based on the other retrieved properties, since only 11894 entries (11%) carry an associated XLogP3 value.: [evidence span withheld — anchor status: not_found]
- [discussion] rotatable bonds (RB) and predicted octanol/water partition coefficient (XLogP3), were obtained from PubChem [51] for all metabolites in the MNXref database: "rotatable bonds (RB) and predicted octanol/water partition coefficient (XLogP3), were obtained from PubChem [51] for all metabolites in the MNXref database"
- [discussion] the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6.: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] If no properties were available, the respective metabolite was classified as not likely to be permeable.: "If no properties were available, the respective metabolite was classified as not likely to be permeable."
- [discussion] so molecular properties for 110266 compounds could be obtained. Since not all entries had an associated XLogP3 value (only 11894: 11%): "so molecular properties for 110266 compounds could be obtained. Since not all entries had an associated XLogP3 value (only 11894: 11%)"
