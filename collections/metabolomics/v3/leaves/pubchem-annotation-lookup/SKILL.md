---
name: pubchem-annotation-lookup
description: Use when a compound list for amanida meta-analysis contains inconsistent
  names/synonyms and each entry must be keyed to one unambiguous compound with PubChem
  CID, InChIKey, KEGG, ChEBI, HMDB, and DrugBank identifiers.
when_to_use_negative:
- The input is raw spectra or a feature table rather than compound-name-level aggregate
  statistics (this skill harmonizes names, not peaks).
- Compounds already carry consistent InChIKeys across all studies, so the 'optimize
  naming combination' workflow is unnecessary.
- No internet/PubChem access is available, since get_cid/pc_prop/cts_convert depend
  on live web service queries.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: webchem
  role: Provides get_cid and pc_prop for PubChem annotation retrieval and cts_convert/pc_synonyms-based
    lookup for InChIKey, KEGG, ChEBI, HMDB, and DrugBank resolution
- name: Easy-Amanida
  role: Shiny app that wraps the harmonization workflow and feeds the harmonized names
    into the amanida meta-analysis
  repo: https://github.com/mariallr/easy-amanida
- name: R
  role: Implementation language in which the harmonization script merges retrieved
    identifiers onto the compound rows
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:5de22e7223f0a9f52d903a897c39dbea2ce5b38e5c2ab6638a62a0c1a814bf7e
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pubchem-annotation-lookup

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes compound names in a metabolomics aggregate-data table by resolving each name to a single PubChem record and retrieving its structural identifiers (CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey) plus KEGG, ChEBI, HMDB, and DrugBank cross-references via the webchem R package, as implemented in Easy-Amanida's naming-harmonization step before meta-analysis.

## When to use

Use when a compound list for amanida meta-analysis contains inconsistent names/synonyms and each entry must be keyed to one unambiguous compound with PubChem CID, InChIKey, KEGG, ChEBI, HMDB, and DrugBank identifiers.

## When NOT to use

- The input is raw spectra or a feature table rather than compound-name-level aggregate statistics (this skill harmonizes names, not peaks).
- Compounds already carry consistent InChIKeys across all studies, so the 'optimize naming combination' workflow is unnecessary.
- No internet/PubChem access is available, since get_cid/pc_prop/cts_convert depend on live web service queries.

## Inputs

- CSV/text table of compound names (with optional aggregate p-value and fold-change statistics) as uploaded to the Easy-Amanida Shiny app

## Outputs

- harmonized_compounds.csv with columns: compound name/id, PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, DrugBank; unmatched names flagged

## How to apply

Load the input table of compound names (with aggregate p-value/fold-change context) into R. Call webchem's get_cid on each name to resolve the PubChem CID, then pc_prop to fetch Molecular Formula, Molecular Weight, Canonical SMILES, and InChIKey for that CID. Use webchem's identifier-conversion functions (cts_convert / pc_synonyms-based lookup) to resolve the KEGG, ChEBI, HMDB, and DrugBank cross-references from the InChIKey or synonyms. Merge every retrieved identifier back onto the original compound-name rows, flagging unmatched or ambiguous names rather than silently dropping them, and write the result to harmonized_compounds.csv. The rationale is the paper's stated need for 'compounds naming harmonization': synonym-rich metabolite names would otherwise split or merge votes during p-value and fold-change combination in amanida.

## Related tools

- **webchem** (Provides get_cid and pc_prop for PubChem annotation retrieval and cts_convert/pc_synonyms-based lookup for InChIKey, KEGG, ChEBI, HMDB, and DrugBank resolution)
- **Easy-Amanida** (Shiny app that wraps the harmonization workflow and feeds the harmonized names into the amanida meta-analysis) — https://github.com/mariallr/easy-amanida
- **R** (Implementation language in which the harmonization script merges retrieved identifiers onto the compound rows)

## Examples

```
cid <- get_cid(name, first = TRUE); props <- pc_prop(cid, properties = c("MolecularFormula","MolecularWeight","CanonicalSMILES","InChIKey"))
```

## Evaluation signals

- harmonized_compounds.csv exists and contains all required identifier columns: PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank.
- Each output row is keyed to a single, unambiguous compound-name-to-identifier combination; no row maps to multiple CIDs or InChIKeys.
- Names that failed PubChem retrieval are flagged (kept and marked) rather than silently removed from the table.
- Retrieved InChIKeys are consistent with the pc_prop SMILES/formula for the same CID (spot-check a few rows).
- After harmonization, synonyms of the same compound collapse to one identifier so amanida votes aggregate per compound rather than per spelling.

## Limitations

- Names with no PubChem hit or with ambiguous synonym matches remain unmatched and cannot contribute to the meta-analysis vote table.
- Identifier retrieval is web-service dependent (PubChem/CTS), so results can vary with service availability and database updates.
- The skill harmonizes naming only; it does not itself perform the p-value/fold-change combination, which is amanida's role.

## Evidence
