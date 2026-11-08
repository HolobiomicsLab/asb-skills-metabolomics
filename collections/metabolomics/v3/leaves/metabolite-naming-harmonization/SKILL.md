---
name: metabolite-naming-harmonization
description: Use when a metabolomics input table contains compound names or IDs with
  synonyms and inconsistent naming that must be resolved to standard cross-database
  identifiers before combining p-values and fold-changes across studies.
when_to_use_negative:
- The input data are raw spectra or feature tables (e.g. mzML peaks) rather than already-named
  compounds with aggregate statistics — Easy-Amanida targets aggregate statistical
  data like p-value and fold-change.
- Compounds are already fully harmonized to a single identifier scheme (e.g. all rows
  keyed by InChIKey) with no synonyms to resolve.
- The goal is a full individual-participant-data meta-analysis rather than combining
  reported aggregate statistics.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: webchem
  role: Provides get_cid/pc_prop for PubChem property retrieval and cts_convert/pc_synonyms
    lookups to resolve InChIKey, KEGG, ChEBI, HMDB, and DrugBank cross-references
  repo: github.com/mariallr/easy-amanida
- name: R
  role: Language in which the harmonization module and Easy-Amanida app are implemented
  repo: github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny app whose naming-harmonization step is implemented by this skill before
    Amanida meta-analysis
  repo: github.com/mariallr/easy-amanida
- name: Shiny
  role: R package add-on used to expose the harmonization workflow as an interactive
    web app
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:114be5fc7e63abea35194dfb349813ca3a6cd294b191b612681a625e86a4f9f5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - Shiny
  - webchem
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabolite-naming-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonize compound names in a user-supplied metabolomics dataset by resolving synonyms to unambiguous database identifiers (PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank) via R's webchem, so that aggregate-statistics meta-analysis in Easy-Amanida/Amanida combines rows referring to the same metabolite.

## When to use

Use when a metabolomics input table contains compound names or IDs with synonyms and inconsistent naming that must be resolved to standard cross-database identifiers before combining p-values and fold-changes across studies.

## When NOT to use

- The input data are raw spectra or feature tables (e.g. mzML peaks) rather than already-named compounds with aggregate statistics — Easy-Amanida targets aggregate statistical data like p-value and fold-change.
- Compounds are already fully harmonized to a single identifier scheme (e.g. all rows keyed by InChIKey) with no synonyms to resolve.
- The goal is a full individual-participant-data meta-analysis rather than combining reported aggregate statistics.

## Inputs

- CSV/table of compound names or IDs from a user-supplied metabolomics dataset (with aggregate statistics such as p-value and fold-change per compound)
- Optional synonym lists for ambiguous compound names

## Outputs

- harmonized_compounds.csv keyed by unambiguous compound name, with columns for PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, and DrugBank identifiers
- Flags for unmatched/ambiguous compound names

## How to apply

Load the table of compound names (with their aggregate statistics context) into R and, for each name, call webchem's PubChem retrieval functions (get_cid followed by pc_prop) to obtain the PubChem CID, Molecular Formula, Molecular Weight, and Canonical SMILES. Then use webchem's identifier-conversion functions (cts_convert or pc_synonyms-based lookup) to resolve the InChIKey plus KEGG, ChEBI, HMDB, and DrugBank cross-references. Merge all retrieved identifiers back onto the original compound-name rows, keeping unmatched names flagged, and write the result to harmonized_compounds.csv. The rationale is that Easy-Amanida's meta-analysis of aggregate p-values and fold-changes requires a workflow that optimizes the naming combination, since compound naming synonyms are a stated barrier to metabolomics meta-analysis.

## Related tools

- **webchem** (Provides get_cid/pc_prop for PubChem property retrieval and cts_convert/pc_synonyms lookups to resolve InChIKey, KEGG, ChEBI, HMDB, and DrugBank cross-references) — github.com/mariallr/easy-amanida
- **R** (Language in which the harmonization module and Easy-Amanida app are implemented) — github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny app whose naming-harmonization step is implemented by this skill before Amanida meta-analysis) — github.com/mariallr/easy-amanida
- **Shiny** (R package add-on used to expose the harmonization workflow as an interactive web app)

## Examples

```
cid <- get_cid("creatinine"); props <- pc_prop(cid$cid, properties = c("MolecularFormula","MolecularWeight","CanonicalSMILES","InChIKey"))
```

## Evaluation signals

- harmonized_compounds.csv exists and contains all required identifier columns (PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank)
- Each output row is keyed to a single unambiguous compound name combination, consistent with the paper's goal of 'ensuring the compounds naming harmonization'
- Unmatched compound names are retained and flagged rather than silently dropped
- Retrieved InChIKeys are structurally consistent with the reported Molecular Formula and Molecular Weight from pc_prop
- Re-running the harmonization on the same input yields a stable identifier mapping (no flapping between synonyms)

## Limitations

- Compound naming synonyms and lack of consensus on statistical reporting are inherent barriers; some names may remain unmatched after PubChem/CTS lookups
- Identifier resolution depends on external services (PubChem, CTS), so availability and lookup coverage limit which compounds can be harmonized
- The discussion section content of the article is absent, so caveats discussed by the authors there could not be extracted

## Evidence

- [methods] harmonization output columns: [evidence span withheld — anchor status: not_found]
- [methods] webchem functions: [evidence span withheld — anchor status: not_found]
