---
name: molecule-identifier-mapping-to-pathway-databases
description: Use when molecular abundance matrices (e.g. metabolite names, ChEBI,
  UniProt or ENSEMBL columns) must be converted to the identifier vocabulary of a
  pathway database (Reactome, KEGG, PathBank) before ssPA scoring or any pathway-based
  multi-omics integration.
when_to_use_negative:
- The abundance matrix columns already carry the database's own identifiers (e.g.
  ChEBI columns for Reactome) — mapping is unnecessary and re-running it risks losing
  valid IDs.
- The analysis does not require pathway transformation, e.g. purely molecular-level
  modelling or ORA/GSEA workflows that take a differential-abundance list and can
  accept the original IDs.
- The molecule cannot be annotated to any pathway in the database — un-annotated molecules
  cannot be incorporated into pathway-based models and will be dropped, so an alternative
  non-pathway approach is preferable.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0091
tools:
- name: sspa
  role: Provides identifier_conversion (download/build a compound name-to-ID conversion
    table) and map_identifiers (relabel matrix columns to the output ID type such
    as ChEBI), plus pathway database loaders that define the target ID scheme
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Consumes the identifier-harmonised multi-omics matrices; requires molecule
    IDs to match Reactome or KEGG pathway IDs before pathway transformation
  repo: https://github.com/cwieder/PathIntegrate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:a6f05f94ecab6544b999e812801d25ed6a3c16d5a18675c2236ba373a73832f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# molecule-identifier-mapping-to-pathway-databases

## Summary

Harmonise metabolite and protein identifiers (metabolite names, KEGG compounds, ChEBI, UniProt, ENSEMBL) so each molecular abundance matrix matches the ID scheme of the chosen pathway database (Reactome/PathBank use ChEBI + UniProt; KEGG uses KEGG compound/gene IDs), a prerequisite for pathway transformation in PathIntegrate.

## When to use

Use when molecular abundance matrices (e.g. metabolite names, ChEBI, UniProt or ENSEMBL columns) must be converted to the identifier vocabulary of a pathway database (Reactome, KEGG, PathBank) before ssPA scoring or any pathway-based multi-omics integration.

## When NOT to use

- The abundance matrix columns already carry the database's own identifiers (e.g. ChEBI columns for Reactome) — mapping is unnecessary and re-running it risks losing valid IDs.
- The analysis does not require pathway transformation, e.g. purely molecular-level modelling or ORA/GSEA workflows that take a differential-abundance list and can accept the original IDs.
- The molecule cannot be annotated to any pathway in the database — un-annotated molecules cannot be incorporated into pathway-based models and will be dropped, so an alternative non-pathway approach is preferable.

## Inputs

- Pandas DataFrame of molecular abundances with samples as rows and molecule identifiers (metabolite names, ChEBI, UniProt, or ENSEMBL) as columns
- Pathway database table from sspa (Reactome, KEGG, or PathBank) or a custom GMT/.csv file
- List of molecule identifier strings to be converted

## Outputs

- Identifier-harmonised abundance matrix whose column names match the chosen pathway database's ID scheme
- Conversion table mapping input molecule names to output ID types (e.g. name to ChEBI)
- Multi-omics pathway DataFrame keyed on the selected identifier types for downstream ssPA transformation

## How to apply

First choose the pathway database and note its ID scheme: Reactome and PathBank pathways are keyed on ChEBI for metabolites and UniProt/ENSEMBL for proteins, while KEGG pathways use KEGG compound and KEGG gene IDs. Collect the molecule identifiers from the columns of each omics DataFrame (samples as rows, molecule IDs as columns) and, where columns carry metabolite names rather than database IDs, download the conversion table with sspa.identifier_conversion (input_type='name') and apply it with sspa.map_identifiers, requesting the output ID type that matches the database (e.g. 'ChEBI'). For multi-omics pathways, download pathway tables from sspa using omics_type='multiomics' and, for Reactome, specify which identifier types ('chebi', 'uniprot', 'gene_symbol') to include so the pathway DataFrame's molecule IDs align with all omics layers. After mapping, verify that a sufficient proportion of molecules carries a valid identifier (ChEBI, UniProt, or ENSEMBL) that hits a pathway, since PathIntegrate discards un-annotated molecules and pathway coverage is a known bottleneck, especially for metabolomics. The resulting harmonised matrices then feed the standardisation and ssPA kPCA/SVD-PLAGE transformation step.

## Related tools

- **sspa** (Provides identifier_conversion (download/build a compound name-to-ID conversion table) and map_identifiers (relabel matrix columns to the output ID type such as ChEBI), plus pathway database loaders that define the target ID scheme) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Consumes the identifier-harmonised multi-omics matrices; requires molecule IDs to match Reactome or KEGG pathway IDs before pathway transformation) — https://github.com/cwieder/PathIntegrate

## Examples

```
conversion_table = sspa.identifier_conversion(input_type="name", compound_list=processed_data.columns.tolist()); mapped = sspa.map_identifiers(conversion_table, output_id_type="ChEBI", matrix=processed_data)
```

## Evaluation signals

- After mapping, column names of each omics matrix belong to the target ID space (ChEBI/UniProt/ENSEMBL for Reactome, KEGG compound/gene for KEGG) rather than metabolite names or mixed vocabularies.
- A high percentage of molecules carries a valid identifier in the target scheme and maps to at least one pathway; the supplementary coverage diagnostics (percentage of molecules with a valid ChEBI, UniProt, or ENSEMBL identifier mapping to Reactome human pathways) should be reproduced for each omics layer.
- The mapped matrix retains the same number of rows (samples) and a consistent sample index, so it can be z-score-standardised column-wise and passed to sspa_kpca or sspa_SVD without ID errors.
- Pathways chosen (single-omics or multi-omics) contain molecules actually observed in the data, and the downstream pathway score matrix has one column per scored pathway with finite values.
- If sspa.map_identifiers is used, the output matrix's column count should match the number of successfully converted compounds from the conversion table; unmatched compounds are dropped and should be counted.

## Limitations

- Pathway coverage — the proportion of molecules in a pathway observable in the data — is an inherent bottleneck, particularly for metabolomics.
- Molecules lacking valid ChEBI/UniProt/ENSEMBL annotations cannot currently be incorporated into pathway-based models, causing information loss.
- KEGG pathways use KEGG compound IDs while Reactome and PathBank use ChEBI/UniProt, so the conversion output type must match the chosen database or IDs will fail to match.

## Evidence

- [readme] sspa identifier conversion functions: "## Identifier harmonization 
```python
# download the conversion table
compound_names = processed_data.columns.tolist()
conversion_table = sspa.identifier_conversion(input_type="name","
- [readme] identifier types per pathway database: [evidence span withheld — anchor status: external_doc]
- [readme] multi-omics pathway download and identifier choice: [evidence span withheld — anchor status: external_doc]
- [methods] molecule ID requirements for PathIntegrate: [evidence span withheld — anchor status: not_found]
- [supplementary] mapping coverage as a validity check: "Percentage of molecules with a valid identifier (ChEBI, UniProt, or ENSEMBL) in single omics mapping to Reactome human pathways"
- [discussion] pathway coverage bottleneck for un-annotated molecules: [evidence span withheld — anchor status: not_found]
- [discussion] information loss from un-annotated molecules: "future work should focus on how un-annotated molecules could be incorporated into pathway-based models, to minimise information loss"
