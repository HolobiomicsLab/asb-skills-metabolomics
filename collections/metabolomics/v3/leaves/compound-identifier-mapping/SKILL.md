---
name: compound-identifier-mapping
description: Use when a user-supplied compound list uses heterogeneous names or synonyms
  and must be mapped to standardized cross-database identifiers (PubChem CID, InChIKey,
  KEGG, ChEBI, HMDB, DrugBank) before downstream analysis.
when_to_use_negative:
- The input is already a harmonized table keyed by InChIKey or PubChem CID with no
  naming ambiguity.
- The input is raw spectral data (e.g. mzML peak lists) rather than a list of compound
  names — identifier mapping is a post-annotation step in this workflow.
- Compounds lack PubChem/CTS representation and are expected to remain unmatched,
  making mapping uninformative.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3331
- http://edamontology.org/topic_3315
tools:
- name: webchem
  role: Provides get_cid, pc_prop, cts_convert, and pc_synonyms functions to retrieve
    PubChem CIDs, properties, and cross-reference identifiers (InChIKey, KEGG, ChEBI,
    HMDB, DrugBank) from compound names
  repo: https://github.com/mariallr/easy-amanida
- name: R
  role: Language in which the harmonization workflow is implemented and in which identifier
    columns are merged back onto the input rows
- name: Easy-Amanida
  role: Shiny app that wraps the amanida + webchem harmonization step and provides
    a workflow to optimize the naming combination
  repo: https://github.com/mariallr/easy-amanida
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:6b78637d4806c4c250f074cce6a8bf79041fca86d7e3859c4e4377e4b14738a1
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3331
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# compound-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes compound names in a metabolomics dataset by resolving each name to standard cross-reference identifiers (PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, DrugBank) using the R package webchem, as implemented in the Easy-Amanida workflow. This ensures unambiguous compound naming before meta-analysis of aggregate statistics.

## When to use

Use when a user-supplied compound list uses heterogeneous names or synonyms and must be mapped to standardized cross-database identifiers (PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank) before downstream analysis.

## When NOT to use

- The input is already a harmonized table keyed by InChIKey or PubChem CID with no naming ambiguity.
- The input is raw spectral data (e.g. mzML peak lists) rather than a list of compound names — identifier mapping is a post-annotation step in this workflow.
- Compounds lack PubChem/CTS representation and are expected to remain unmatched, making mapping uninformative.

## Inputs

- User-supplied table of compound names (optionally with aggregate statistics such as p-value and fold-change)
- R session with webchem installed

## Outputs

- harmonized_compounds.csv with one row per compound name and columns for PubChem CID, Molecular Formula, Molecular Weight, Canonical SMILES, InChIKey, KEGG, ChEBI, HMDB, and DrugBank identifiers
- Flagged/annotated entries for compound names that could not be matched to a unique identifier

## How to apply

Load the input table of compound names into R. For each name, call webchem's PubChem retrieval functions (get_cid followed by pc_prop) to obtain the PubChem CID, Molecular Formula, Molecular Weight, and Canonical SMILES. Then use webchem's identifier-conversion functions (cts_convert or pc_synonyms-based lookup) to resolve the InChIKey and the KEGG, ChEBI, HMDB, and DrugBank cross-references. Merge all retrieved identifiers back onto the original compound-name rows, flagging unmatched names rather than dropping them, and write the result to harmonized_compounds.csv. The rationale, per the Easy-Amanida paper, is that 'compound naming synonyms' and lack of consensus in statistical reporting hinder metabolomics meta-analysis, so harmonization must precede the meta-analysis of aggregate statistics (p-value, fold-change).

## Related tools

- **webchem** (Provides get_cid, pc_prop, cts_convert, and pc_synonyms functions to retrieve PubChem CIDs, properties, and cross-reference identifiers (InChIKey, KEGG, ChEBI, HMDB, DrugBank) from compound names) — https://github.com/mariallr/easy-amanida
- **R** (Language in which the harmonization workflow is implemented and in which identifier columns are merged back onto the input rows)
- **Easy-Amanida** (Shiny app that wraps the amanida + webchem harmonization step and provides a workflow to optimize the naming combination) — https://github.com/mariallr/easy-amanida

## Examples

```
cid <- get_cid(compound_names); props <- pc_prop(cid$cid, properties = c("MolecularFormula","MolecularWeight","CanonicalSMILES","InChIKey"))
```

## Evaluation signals

- harmonized_compounds.csv exists and contains all required identifier columns: PubChem CID, InChIKey, KEGG, ChEBI, HMDB, DrugBank.
- Each row is keyed to a single, unambiguous compound-name-to-identifier combination.
- Unmatched compound names are explicitly flagged rather than silently dropped from the output table.
- Retrieved Molecular Formula / Molecular Weight / Canonical SMILES are internally consistent with the resolved PubChem CID.
- The InChIKey resolution agrees with the PubChem CID for each compound (no mixed-source mismatches).

## Limitations

- Compound-name synonyms are a root cause of failed or ambiguous matching; names not present in PubChem or CTS remain flagged/unmatched.
- The article's discussion section content is absent from the available context, so additional caveats documented there could not be extracted.
- Harmonization quality depends on the upstream user table; the workflow 'provides a workflow to optimize the naming combination' but ambiguous input names can still map to multiple candidates.

## Evidence
