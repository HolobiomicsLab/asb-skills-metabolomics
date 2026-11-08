---
name: cross-database-identifier-retrieval
description: Use when a metabolomics input table lists compounds under inconsistent
  names or synonyms and the rows must be keyed to unambiguous database identifiers
  (CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank) before downstream aggregation or meta-analysis.
when_to_use_negative:
- The input rows already carry unambiguous identifiers (e.g. InChIKey or CID) for
  every compound, so no name-to-identifier resolution is needed
- The analysis goal is raw-signal processing or feature detection rather than harmonizing
  compound naming prior to meta-analysis of aggregate statistics
- Compounds are not chemical entities resolvable in PubChem (e.g. purely descriptive
  clinical variables)
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: webchem
  role: Provides PubChem retrieval (get_cid, pc_prop) and identifier-conversion (cts_convert,
    pc_synonyms) functions used to resolve CID, formula, MW, SMILES, InChIKey, and
    KEGG/ChEBI/HMDB/DrugBank cross-references
  repo: https://github.com/mariallr/easy-amanida
- name: R
  role: Language in which the harmonization module is implemented and run
- name: Easy-Amanida
  role: Shiny app wrapping the harmonization workflow that optimizes the naming combination
    before meta-analysis
  repo: https://github.com/mariallr/easy-amanida
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:b1ed0522c2fa5f1fe89a7cc6338836ffd2307a825e8f91dbdb888908b81dfec2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  repo_url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - webchem
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-database-identifier-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes compound names in a user-supplied metabolomics dataset by retrieving cross-database identifiers (PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, DrugBank) using webchem's PubChem retrieval and identifier-conversion functions in R.

## When to use

Use when a metabolomics input table lists compounds under inconsistent names or synonyms and the rows must be keyed to unambiguous database identifiers (CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank) before downstream aggregation or meta-analysis.

## When NOT to use

- The input rows already carry unambiguous identifiers (e.g. InChIKey or CID) for every compound, so no name-to-identifier resolution is needed
- The analysis goal is raw-signal processing or feature detection rather than harmonizing compound naming prior to meta-analysis of aggregate statistics
- Compounds are not chemical entities resolvable in PubChem (e.g. purely descriptive clinical variables)

## Inputs

- Input table of compound names with aggregate statistics context (e.g. p-value, fold-change rows) in R
- List of compound names or ids requiring harmonization

## Outputs

- harmonized_compounds.csv containing compound name/id, PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, and DrugBank identifier columns
- Flagged entries for unmatched compound names

## How to apply

Load the input table of compound names (with aggregate statistics context) into R, then call webchem's PubChem retrieval functions — get_cid followed by pc_prop — to obtain the PubChem CID, Molecular Formula, Molecular Weight, and Canonical SMILES for each compound name. Next, use webchem's identifier-conversion functions (e.g. cts_convert / pc_synonyms-based lookup) to resolve the InChIKey and the KEGG, ChEBI, HMDB, and DrugBank cross-references for each compound. Merge all retrieved identifiers back onto the original compound name rows, keeping unmatched names flagged rather than dropped, and write the harmonized table to harmonized_compounds.csv. The rationale, per Easy-Amanida, is that compound naming synonyms and lack of naming consensus hinder metabolomics meta-analysis, so harmonization must precede combining p-value and fold-change aggregate statistics.

## Related tools

- **webchem** (Provides PubChem retrieval (get_cid, pc_prop) and identifier-conversion (cts_convert, pc_synonyms) functions used to resolve CID, formula, MW, SMILES, InChIKey, and KEGG/ChEBI/HMDB/DrugBank cross-references) — https://github.com/mariallr/easy-amanida
- **R** (Language in which the harmonization module is implemented and run)
- **Easy-Amanida** (Shiny app wrapping the harmonization workflow that optimizes the naming combination before meta-analysis) — https://github.com/mariallr/easy-amanida

## Examples

```
cid <- get_cid(compound_name); props <- pc_prop(cid, properties = c('MolecularFormula','MolecularWeight','CanonicalSMILES','InChIKey')); kegg <- cts_convert(compound_name, from='PubChem CID', to=c('KEGG','ChEBI','HMDB','DrugBank'))
```

## Evaluation signals

- harmonized_compounds.csv exists and contains all required identifier columns: PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank
- Each output row is keyed to a single unambiguous compound name combination, consistent with the paper's goal of 'ensuring the compounds naming harmonization'
- Retrieved Molecular Formula, Molecular Weight, and Canonical SMILES are non-empty and internally consistent per PubChem CID
- Unmatched compound names are flagged in the output rather than silently dropped
- Rows of the harmonized table map one-to-one onto the original compound name rows of the input table

## Limitations

- Compound names without a PubChem match remain unresolved and must be flagged; naming synonyms and lack of statistical-reporting consensus are exactly the obstacles this step addresses
- Identifier retrieval depends on external PubChem/CTS services, so resolution quality tracks those databases' coverage of the queried compound names
- Discussion-section content for Easy-Amanida is absent in the source document, so long-term maintenance caveats could not be extracted

## Evidence

- [methods] The harmonization step produces a final output table containing the compound name or id, Pub Chem CID, Molecular Formula, Molecular Weight, Canonical SMILES, In ChIKey, KEGG, ChEBI, HMDB, and Drugbank identifiers.: [evidence span withheld — anchor status: not_found]
