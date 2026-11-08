---
name: gap-filling-database-annotation
description: Use when preparing a MNXref-based gap-filling database for COMMIT community
  gap filling and you need to tag transport reactions with metabolite permeability
  to reduce their cost.
when_to_use_negative:
- The gap-filling database uses non-MNXref identifiers for which no chem_prop.tsv/chem_xref.tsv
  translation to PubChem is available, so no molecular properties can be retrieved.
- The gap-filling strategy does not weight transport reactions by permeability (e.g.
  a plain individual-model gap fill), making the annotation superfluous.
- Only a handful of metabolites need manual permeability judgment rather than database-wide
  annotation of the MNXref metabolite set.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_2258
tools:
- name: COMMIT
  role: Provides addNamesPermeabilityToGfDb to attach per-metabolite permeability
    classes and build the reduced-cost transport-reaction subset of the gap-filling
    database
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox v3.0
  role: Matlab dependency within which COMMIT and the gap-filling database (.mat)
    are handled
  repo: https://github.com/opencobra/cobratoolbox
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:62bd09a1244ecec0e825017cfa3202a4f549179fe312e54b31ee93342343e1f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gap-filling-database-annotation

## Summary

Annotates the COMMIT gap-filling universal database (Universal-model-MNXref-balanced.mat) with per-metabolite permeability classes derived from PubChem molecular properties and Lipinski-style thresholds, so that permeable transport reactions receive reduced gap-filling costs.

## When to use

Use when preparing a MNXref-based gap-filling database for COMMIT community gap filling and you need to tag transport reactions with metabolite permeability to reduce their cost.

## When NOT to use

- The gap-filling database uses non-MNXref identifiers for which no chem_prop.tsv/chem_xref.tsv translation to PubChem is available, so no molecular properties can be retrieved.
- The gap-filling strategy does not weight transport reactions by permeability (e.g. a plain individual-model gap fill), making the annotation superfluous.
- Only a handful of metabolites need manual permeability judgment rather than database-wide annotation of the MNXref metabolite set.

## Inputs

- MNXref metabolite identifiers (MNXref database)
- MNXref translation tables chem_prop.tsv and chem_xref.tsv
- PubChem molecular properties: MW, TPSA, HBD, HBA, RB, XLogP3
- Balanced gap-filling database Universal-model-MNXref-balanced.mat

## Outputs

- Per-metabolite permeability classification table
- Reduced-cost (permeable) transport-reaction subset added to the gap-filling database (data/gap-filling/database/Universal-model-MNXref-balanced.mat)
- Set of candidate reduced-cost transport reactions (2534 of 4520 in the reference run)

## How to apply

Map MNXref metabolite identifiers to PubChem entries using the MNXref translation tables (chem_prop.tsv, chem_xref.tsv), noting that only ~26% of InChI keys translate to PubChem CIDs (molecular properties for 110266 compounds retrievable). Retrieve six molecular properties from PubChem: molecular weight (MW), topological polar surface area (TPSA), hydrogen bond donors (HBD), hydrogen bond acceptors (HBA), rotatable bonds (RB) and predicted octanol/water partition coefficient (XLogP3). Impute missing XLogP3 values (only 11894 entries, 11%, carry one) with kNN based on the other properties. Classify each metabolite as likely permeable if HBD<=5, HBA<=10, MW<=500 Da, TPSA<=140 A, RB<=10 and -0.4<=XLogP3<=5.6; metabolites without available properties are classified as not likely permeable. Write the per-metabolite permeability table and build the reduced-cost (permeable) transport-reaction subset via the COMMIT function addNamesPermeabilityToGfDb(dbModel_MNXref_balanced, mnxref_dir), which marks candidates for reduced cost in the gap-filling database — in this study 2534 out of 4520 transport reactions.

## Related tools

- **COMMIT** (Provides addNamesPermeabilityToGfDb to attach per-metabolite permeability classes and build the reduced-cost transport-reaction subset of the gap-filling database) — https://github.com/pwendering/COMMIT
- **COBRA toolbox v3.0** (Matlab dependency within which COMMIT and the gap-filling database (.mat) are handled) — https://github.com/opencobra/cobratoolbox

## Examples

```
addNamesPermeabilityToGfDb(dbModel_MNXref_balanced, mnxref_dir)
```

## Evaluation signals

- Every MNXref metabolite in the output table carries a permeability classification consistent with the six thresholds (HBD<=5, HBA<=10, MW<=500 Da, TPSA<=140 A, RB<=10, -0.4<=XLogP3<=5.6).
- Metabolites without available molecular properties are marked as not likely permeable rather than left unclassified.
- The count of reduced-cost (permeable) transport-reaction candidates is a reproducible fraction of all transport reactions (reference run: 2534 of 4520).
- The annotated database file data/gap-filling/database/Universal-model-MNXref-balanced.mat loads in the COBRA toolbox and includes the permeability-based reduced-cost annotations.
- Imputed XLogP3 entries are traceable to the kNN imputation step given that only ~11% of entries carry an original XLogP3 value.

## Limitations

- Only ~26% of InChI keys translate to PubChem CIDs, so molecular properties could be obtained for only 110266 compounds; the remainder default to not likely permeable.
- Only 11894 entries (11%) have an associated XLogP3 value, making the classification dependent on kNN imputation for most metabolites.
- The exact Lipinski-style threshold set (e.g. TPSA<=140 A, -0.4<=XLogP3<=5.6) was applied as-is; alternative permeability criteria are not evaluated in the study.
- Full COMMIT procedure specification is deferred to supplementary S1 Text rather than the main text.

## Evidence

- [methods] lipinski thresholds: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [methods] missing properties: "If no properties were available, the respective metabolite was classified as not likely to be permeable."
- [discussion] pubchem coverage: "so molecular properties for 110266 compounds could be obtained. Since not all entries had an associated XLogP3 value (only 11894: 11%)"
- [results] permeability source: "rotatable bonds (RB) and predicted octanol/water partition coefficient (XLogP3), were obtained from PubChem [51] for all metabolites in the MNXref database"
- [readme] repo role: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions"
