---
name: non-systematic-identifier-mapping
description: Use when metabolite entries from merged studies carry non-systematic
  identifiers (CAS, HMDB, KEGG, etc.) instead of InChIKeys and a systematic, structure-based
  key is needed for dataset merging.
when_to_use_negative:
- Entries already carry an InChIKey — no translation is needed and the row should
  be routed directly to merging.
- The only available input is a metabolite *name* with no identifier; in that case
  the name-to-InChIKey retrieval branch of MMS Step 1 applies instead, not CTS identifier
  translation.
- You need chemical attributes (MW, formula, ontology) rather than a merge key; that
  belongs to MMS Step 2 attribute retrieval, not this translation step.
tools:
- name: Chemical Translation Service (CTS)
  role: REST service called from Python to convert non-systematic identifiers (CAS,
    HMDB, KEGG, etc.) into InChIKeys for the merged table.
- name: Python
  role: Scripting language used to call the CTS REST interface and write the merged
    identifier-to-InChIKey table to disk.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:ee6bb90de6aa3fc80322a45a95395f4e11a336903fa8ea27df1bab0cd53f5762
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translation Service (CTS)
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# non-systematic-identifier-mapping

## Summary

Converts non-systematic metabolite identifiers (CAS, HMDB, KEGG, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID) into InChIKeys via the Chemical Translation Service (CTS) so heterogeneous study datasets can be merged in MMS Step 1.

## When to use

Use when metabolite entries from merged studies carry non-systematic identifiers (CAS, HMDB, KEGG, etc.) instead of InChIKeys and a systematic, structure-based key is needed for dataset merging.

## When NOT to use

- Entries already carry an InChIKey — no translation is needed and the row should be routed directly to merging.
- The only available input is a metabolite *name* with no identifier; in that case the name-to-InChIKey retrieval branch of MMS Step 1 applies instead, not CTS identifier translation.
- You need chemical attributes (MW, formula, ontology) rather than a merge key; that belongs to MMS Step 2 attribute retrieval, not this translation step.

## Inputs

- Metabolite entries whose identifier is a non-systematic ID (CAS, HMDB, KEGG, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID)
- Case-study source tables: Metabolomics Workbench studies ST001039, ST001048, ST001317; HMDB asthma-urine entries; literature articles LT01-06

## Outputs

- Merged identifier-to-InChIKey mapping table with one row per submitted identifier, the resolved InChIKey, and a Reference column retaining the originating source
- Flagged list of identifiers the CTS failed to convert, forwarded to the MMS Step 2 missing-attributes retrieval

## How to apply

In MMS Step 1, first separate the entries whose reported identifier is not already an InChIKey from those that are. For the non-systematic entries, call the Chemical Translation Service (CTS) through its REST interface from Python, submitting each identifier (e.g. CAS number, HMDB ID, KEGG ID, PubChem ID, ChEBI ID, LipidMaps ID, DrugBank ID, Bin ID). Record the identifier's originating source in a Reference column, as the MMS prescribes, then merge the CTS-returned InChIKeys with the InChIKeys retrieved from metabolite names. The rationale is that InChIKeys are the systematic, structure-derived keys that enable merging across sources (Metabolomics Workbench studies ST001039/ST001048/ST001317, HMDB asthma-urine entries, and literature articles LT01-06), whereas the raw non-systematic IDs cannot be joined unambiguously.

## Related tools

- **Chemical Translation Service (CTS)** (REST service called from Python to convert non-systematic identifiers (CAS, HMDB, KEGG, etc.) into InChIKeys for the merged table.)
- **Python** (Scripting language used to call the CTS REST interface and write the merged identifier-to-InChIKey table to disk.)

## Examples

```
python translate_ids.py --input metabolites.tsv --id-column id --output inchikey_map.tsv
```

## Evaluation signals

- Output mapping table exists and contains one row per submitted non-systematic identifier with a populated InChIKey.
- The Reference column retains the originating source for every row, as the MMS prescribes.
- Identifiers that CTS could not convert are explicitly flagged rather than silently dropped, and are routed to the MMS Step 2 missing-attributes retrieval.
- The translated InChIKeys merge cleanly with the InChIKeys obtained from the name-retrieval branch (no duplicate rows for the same metabolite).

## Limitations

- CTS may fail to convert some non-systematic identifiers, leaving gaps that must be handled by the downstream MMS Step 2 missing-attributes retrieval.
- The skill covers identifier-to-InChIKey translation only; it does not itself perform the full three-step MMS curation (conjugate base/acid repair, missing attributes, synonym checking).

## Evidence

- [abstract] Non-systematic identifier translation via CTS: "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [abstract] MMS Step 1 translation and merging: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
