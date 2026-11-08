---
name: chemical-standardization-inchikey-resolution
description: Use when a user-supplied table of compound names (with aggregate p-value/fold-change
  statistics) contains naming synonyms or ambiguous identifiers that must be harmonized
  to a single identifier scheme before meta-analysis.
when_to_use_negative:
- Input compounds already carry consistent InChIKeys or other unambiguous identifiers
  and need no name-based lookup
- The analysis goal is the meta-analysis itself (p-value/fold-change combination)
  rather than pre-analysis identifier harmonization — use the amanida meta-analysis
  step instead
- Compounds are not indexable by name in PubChem or the conversion services (e.g.
  unnamed features, isotopologues without CAS/name entries)
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3292
tools:
- name: webchem
  role: Provides PubChem retrieval (get_cid, pc_prop) and identifier-conversion (cts_convert,
    pc_synonyms) functions to resolve CID, formula, MW, SMILES, InChIKey, and cross-database
    IDs
  repo: github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny app embedding this harmonization step ahead of the amanida meta-analysis;
    supplies the workflow to optimize naming combination
  repo: github.com/mariallr/easy-amanida
- name: R
  role: Runtime language in which webchem lookups are executed and the harmonized
    table is assembled and written to CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:281742d4eaa49981292c0b56cd6f36f0639dd1f29e225892c37555bdcff0bee6
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
  - webchem
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chemical-standardization-inchikey-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes compound names in a metabolomics dataset by resolving each name to unambiguous chemical identifiers — PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, and KEGG/ChEBI/HMDB/DrugBank cross-references — using webchem's PubChem retrieval and identifier-conversion functions, a prerequisite for valid compound-name combination in the Easy-Amanida meta-analysis.

## When to use

Use when a user-supplied table of compound names (with aggregate p-value/fold-change statistics) contains naming synonyms or ambiguous identifiers that must be harmonized to a single identifier scheme before meta-analysis.

## When NOT to use

- Input compounds already carry consistent InChIKeys or other unambiguous identifiers and need no name-based lookup
- The analysis goal is the meta-analysis itself (p-value/fold-change combination) rather than pre-analysis identifier harmonization — use the amanida meta-analysis step instead
- Compounds are not indexable by name in PubChem or the conversion services (e.g. unnamed features, isotopologues without CAS/name entries)

## Inputs

- User-supplied table of compound names with aggregate statistics context (e.g. p-value, fold-change rows)
- Compound name or id column requiring harmonization to cross-database identifiers

## Outputs

- harmonized_compounds.csv containing compound name/id, PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, and DrugBank identifiers
- Flagged rows for compound names that could not be matched unambiguously

## How to apply

Load the input table of compound names into R and, per name, call webchem's PubChem retrieval functions (get_cid followed by pc_prop) to obtain the PubChem CID, Molecular Formula, Molecular Weight, and Canonical SMILES. Then use webchem's identifier-conversion functions (cts_convert / pc_synonyms-based lookup) to resolve the InChIKey and the KEGG, ChEBI, HMDB, and DrugBank cross-references for each compound. The decision point is whether every retrieved identifier maps back to a single unambiguous compound-name combination: unmatched names must be flagged rather than silently dropped, since the paper's goal is 'ensuring the compounds naming harmonization'. Merge all retrieved identifiers back onto the original compound-name rows and write the result to harmonized_compounds.csv; this harmonized table is what feeds the downstream amanida meta-analysis of p-values and fold-changes.

## Related tools

- **webchem** (Provides PubChem retrieval (get_cid, pc_prop) and identifier-conversion (cts_convert, pc_synonyms) functions to resolve CID, formula, MW, SMILES, InChIKey, and cross-database IDs) — github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny app embedding this harmonization step ahead of the amanida meta-analysis; supplies the workflow to optimize naming combination) — github.com/mariallr/easy-amanida
- **R** (Runtime language in which webchem lookups are executed and the harmonized table is assembled and written to CSV)

## Examples

```
cid <- get_cid(compound_name); props <- pc_prop(cid, properties = c('MolecularFormula','MolecularWeight','CanonicalSMILES','InChIKey'))
```

## Evaluation signals

- harmonized_compounds.csv exists and contains all required identifier columns: PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank
- Each row is keyed to a single unambiguous compound-name combination (no duplicate or conflicting identifier assignments per name)
- Unmatched compound names are flagged rather than silently omitted from the output table
- InChIKey standard (27-character, three-block) structure is consistent with the Canonical SMILES and Molecular Formula retrieved via pc_prop

## Limitations

- Compound names must be resolvable via PubChem (get_cid) or the conversion services; synonyms and non-standard naming are the exact failure mode this step mitigates, so unresolved names will remain flagged
- Identifier resolution depends on external web services (PubChem/CTS), so results can vary with database updates or service availability
- Discussion section content is absent from the source document, so limitations are inferred from the methods/abstract rather than an explicit changelog

## Evidence

- [methods] The harmonization step produces a final output table containing the compound name or id, Pub Chem CID, Molecular Formula, Molecular Weight, Canonical SMILES, In ChIKey, KEGG, ChEBI, HMDB, and Drugbank identifiers.: [evidence span withheld — anchor status: not_found]
- [readme] combines two R packages, `amanida` and `webchem`: "combines two R packages, `amanida` and `webchem`, to enable meta-analysis of aggregate statistical data, like p-value and fold-change, while ensuring the compounds naming harmonization."
- [abstract] provides a workflow to optimize naming combination: "The Easy-Amanida app is implemented in Shiny, an R package add-on for interactive web apps, and provides a workflow to optimize naming combination."
