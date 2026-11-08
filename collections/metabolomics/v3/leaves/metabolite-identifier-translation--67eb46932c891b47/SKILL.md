---
name: metabolite-identifier-translation
description: Use when metabolite entries from different studies report identifiers
  other than InChIKeys (e.g. CAS, HMDB ID, KEGG ID) and the datasets must be merged
  or intercompared on a common structural key.
when_to_use_negative:
- The reported metabolite identifier is already an InChIKey — the translation branch
  only handles non-InChIKey identifiers
- You are executing MMS Step 2 attribute retrieval (RefMet name, SMILES, BinVestigate
  Bin ID, ClassyFire ontology) rather than Step 1 identifier harmonization
- The datasets are already merged on InChIKey and only curation steps (conjugate conversion,
  synonym checking) remain
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Chemical Translation Service (CTS)
  role: REST-based converter of non-systematic identifiers (CAS, HMDB ID, KEGG ID,
    etc.) into InChIKeys during MMS Step 1; also supplies molecular weight, formula,
    and IDs in Step 2
- name: PubChem Identifier Exchange Service
  role: Translates metabolite names to InChIKeys; its results are merged with the
    CTS-translated InChIKeys before merging datasets
- name: Python
  role: Scripting language used to call the CTS REST interface and write the merged
    identifier-to-InChIKey mapping table to disk
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:20030abecc33c3efc711d3e5b11f12a18baaf3c6e51b8d015c9155b0e4c02608
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinVestigate
  - ChEBI
  - Chemical Translation Service (CTS)
  - ClassyFire
  - DrugBank
  - HMDB
  - KEGG
  - LipidMaps
  - Metabolomics Workbench
  - PubChem Identifier Exchange Service
  - Python
  - RefMet
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# metabolite-identifier-translation

## Summary

Translates metabolite entries that carry non-InChIKey identifiers (CAS, HMDB, KEGG, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID, CAS numbers) into InChIKeys via the Chemical Translation Service (CTS) so heterogeneous datasets can be merged on a common key, as the first branch of MMS Step 1.

## When to use

Use when metabolite entries from different studies report identifiers other than InChIKeys (e.g. CAS, HMDB ID, KEGG ID) and the datasets must be merged or intercompared on a common structural key.

## When NOT to use

- The reported metabolite identifier is already an InChIKey — the translation branch only handles non-InChIKey identifiers
- You are executing MMS Step 2 attribute retrieval (RefMet name, SMILES, BinVestigate Bin ID, ClassyFire ontology) rather than Step 1 identifier harmonization
- The datasets are already merged on InChIKey and only curation steps (conjugate conversion, synonym checking) remain

## Inputs

- Metabolite entries with non-InChIKey identifiers (CAS, HMDB ID, KEGG ID, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID) from Metabolomics Workbench studies ST001039/ST001048/ST001317, HMDB asthma-urine entries, and literature articles LT01-06
- Metabolite names requiring name-to-InChIKey translation via the PubChem Identifier Exchange Service

## Outputs

- Merged identifier-to-InChIKey mapping table (one row per submitted identifier, InChIKey populated, originating source recorded in a Reference column)
- List of identifiers the CTS failed to convert, flagged for the MMS Step 3b missing-attributes retrieval

## How to apply

Load the metabolite entries from the case-study sources (Metabolomics Workbench studies ST001039/ST001048/ST001317, HMDB asthma-urine entries, and six literature articles LT01-06) and select those whose reported identifier is not already an InChIKey. For each such entry, call the Chemical Translation Service (CTS) via its REST interface from Python to convert the non-systematic identifier (e.g. CAS, HMDB ID, KEGG ID) into an InChIKey; metabolite names are translated separately via the PubChem Identifier Exchange Service. Merge the returned InChIKeys with the original identifier values, recording the originating source in a 'Reference' column as the MMS prescribes, and write the merged identifier-to-InChIKey mapping table to disk for subsequent MMS steps. Any identifiers the CTS fails to convert are flagged and deferred to the Step 3b missing-attributes retrieval, where PubChem and CTS are re-queried. This translation branch is the prerequisite for the duplicate-combining merge on InChIKey that must reach 547 pre-curation rows (391 after full curation) in the urinary asthma demonstration.

## Related tools

- **Chemical Translation Service (CTS)** (REST-based converter of non-systematic identifiers (CAS, HMDB ID, KEGG ID, etc.) into InChIKeys during MMS Step 1; also supplies molecular weight, formula, and IDs in Step 2)
- **PubChem Identifier Exchange Service** (Translates metabolite names to InChIKeys; its results are merged with the CTS-translated InChIKeys before merging datasets)
- **Python** (Scripting language used to call the CTS REST interface and write the merged identifier-to-InChIKey mapping table to disk)

## Examples

```
import requests; r = requests.get("http://cts.fiehnlab.ucdavis.edu/service/convert/" + id_type + "/" + id_value); inchikey = r.json()[0]["result"][0]
```

## Evaluation signals

- The output mapping table exists and contains one row per submitted identifier with an InChIKey populated
- Source provenance is retained for every translated entry in the Reference column
- Pre-curation merged table contains exactly 547 metabolite rows for the ten urinary asthma sources (final curated dataset: 391 rows in merged_metabolites.csv)
- Identifiers the CTS fails to convert are flagged and carried into the Step 3b missing-attributes re-query rather than dropped silently

## Limitations

- The CTS does not convert every non-systematic identifier; unconverted entries must be flagged for the later missing-attributes retrieval step (MMS Step 3b, re-querying PubChem and CTS)
- The translation branch only applies to entries lacking InChIKeys; entries already carrying InChIKeys bypass it

## Evidence

- [abstract] non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number): "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [abstract] Step 1 translation and merging: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [abstract] MMS harmonization rationale: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
