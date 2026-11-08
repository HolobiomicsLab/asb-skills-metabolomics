---
name: mnxref-identifier-cross-referencing
description: Use when draft reconstructions from different pipelines (KBase, RAVEN
  2.0, CarveMe, AuReMe/Pathway Tools) use incompatible identifier namespaces, or when
  molecular properties must be pulled from PubChem for MNXref metabolites.
when_to_use_negative:
- When reconstructions already share a common identifier namespace (e.g., all models
  are already MNXref-consistent) and no external molecular properties are needed
- When the analysis requires full compound coverage — only ~26% of InChI keys translate
  to PubChem CIDs, so properties will be missing for most metabolites
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3316
tools:
- name: COMMIT
  role: Consumes the MNXref-to-PubChem mapping to build the per-metabolite permeability
    table via addNamesPermeabilityToGfDb(dbModel_MNXref_balanced, mnxref_dir)
  repo: https://github.com/pwendering/COMMIT
- name: MetaNetX/MNXref
  role: Provides the reference translation files (chem_prop.tsv, chem_xref.tsv) used
    to unify metabolite and reaction identifiers across reconstructions
- name: PubChem
  role: Source of the six molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) retrieved
    for mapped MNXref metabolites
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:59aa83b5f01983e84951edc8140042ce5dac52aa43b9fa72efa99ea59d0359e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COMMIT
  - MetaNetX/MNXref
  - PubChem
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mnxref-identifier-cross-referencing

## Summary

Translates metabolite and reaction identifiers to MNXref using the MetaNetX translation tables (chem_prop.tsv, chem_xref.tsv) and maps MNXref entries to PubChem CIDs via InChI keys, enabling retrieval of molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) for downstream permeability classification.

## When to use

Use when draft reconstructions from different pipelines (KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools) use incompatible identifier namespaces, or when molecular properties must be pulled from PubChem for MNXref metabolites.

## When NOT to use

- When reconstructions already share a common identifier namespace (e.g., all models are already MNXref-consistent) and no external molecular properties are needed
- When the analysis requires full compound coverage — only ~26% of InChI keys translate to PubChem CIDs, so properties will be missing for most metabolites

## Inputs

- Draft genome-scale metabolic reconstructions with self-defined identifier namespaces (e.g., from KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools)
- MNXref translation tables chem_prop.tsv and chem_xref.tsv from the MetaNetX database
- PubChem molecular property records (MW, TPSA, HBD, HBA, RB, XLogP3) keyed by PubChem CID

## Outputs

- Per-metabolite table of MNXref IDs with translated PubChem CIDs and six molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3), including kNN-imputed XLogP3 values
- Permeability classification per metabolite (likely / not likely permeable) feeding the reduced-cost transport-reaction subset

## How to apply

Translate reaction and metabolite identifiers to MNXref using the MNXref reference files (chem_prop.tsv, chem_xref.tsv), replacing conflicting IDs such as MNXM01 with MNXM1 (both encode H+) and ignoring generic compartments. Map MNXref InChI keys to PubChem CIDs using the MetaNetX translation tables, noting that only ~26% of InChI keys translate, yielding properties for 110266 compounds. Retrieve six molecular properties from PubChem — molecular weight (MW), topological polar surface area (TPSA), hydrogen bond donors (HBD), hydrogen bond acceptors (HBA), rotatable bonds (RB), and predicted octanol/water partition coefficient (XLogP3). Because only 11894 entries (11%) carry an associated XLogP3 value, impute missing XLogP3 by kNN based on the other retrieved properties. Metabolites with no available properties are flagged downstream as not likely permeable, so the mapping coverage directly determines the usable compound set.

## Related tools

- **COMMIT** (Consumes the MNXref-to-PubChem mapping to build the per-metabolite permeability table via addNamesPermeabilityToGfDb(dbModel_MNXref_balanced, mnxref_dir)) — https://github.com/pwendering/COMMIT
- **MetaNetX/MNXref** (Provides the reference translation files (chem_prop.tsv, chem_xref.tsv) used to unify metabolite and reaction identifiers across reconstructions)
- **PubChem** (Source of the six molecular properties (MW, TPSA, HBD, HBA, RB, XLogP3) retrieved for mapped MNXref metabolites)

## Examples

```
addNamesPermeabilityToGfDb(dbModel_MNXref_balanced, mnxref_dir)
```

## Evaluation signals

- Every MNXref metabolite in the output table carries a permeability classification consistent with the six thresholds (HBD<=5, HBA<=10, MW<=500 Da, TPSA<=140 A, RB<=10, -0.4<=XLogP3<=5.6)
- Metabolites without available PubChem properties are marked as not likely permeable rather than left unclassified
- Mapping coverage should approximate the reported ~26% InChI-key-to-CID translation rate (~110266 compounds with properties)
- No metabolite retains the conflicting ID MNXM01 after translation (replaced with MNXM1, both encoding H+)
- Imputed XLogP3 values fall within the plausibility range used by the permeability filter (-0.4 to 5.6 applies to all classified metabolites)

## Limitations

- Only ~26% of InChI keys translate to PubChem CIDs, so molecular properties exist for only 110266 of the MNXref compounds
- Only 11% (11894) of entries carry an associated XLogP3 value, requiring kNN imputation based on the other properties
- Comparison to reference models is hampered by lack of reaction cross-references to other namespaces; translations are only done 'whenever possible'

## Evidence

- [methods] MNXref translation tables: [evidence span withheld — anchor status: not_found]
- [discussion] MNXref reference files: "Reaction and metabolite identifiers were translated to MNXref identifiers using the MNXref reference files for metabolites and reactions."
- [discussion] PubChem property coverage: "so molecular properties for 110266 compounds could be obtained. Since not all entries had an associated XLogP3 value (only 11894: 11%)"
- [discussion] MNXM01 replacement: "the metabolite ID ’MNXM01’ was replaced with ’MNXM1’ since both encode H+."
- [methods] Imputation and rule application: [evidence span withheld — anchor status: not_found]
