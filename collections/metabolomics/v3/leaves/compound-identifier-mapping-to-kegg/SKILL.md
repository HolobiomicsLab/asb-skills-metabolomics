---
name: compound-identifier-mapping-to-kegg
description: Use when an untargeted metabolomics dataset holds compound names that
  must be mapped to KEGG compound identifiers before ORA against KEGG release 96 human
  pathway sets.
when_to_use_negative:
- The dataset already uses KEGG identifiers natively — remapping through MetaboAnalyst
  would add conversion noise without benefit.
- The downstream pathway database is Reactome or BioCyc but only KEGG identifiers
  are needed — map to ChEBI/BioCyc identifiers instead of forcing KEGG IDs.
- The analysis goal is a comparison of pathway databases rather than KEGG-specific
  ORA — the identifier harmonisation step must span all three databases (KEGG, Reactome,
  BioCyc), not just KEGG.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3391
tools:
- name: MetaboAnalyst 4.0
  role: ID conversion tool used to convert compound names to KEGG identifiers for
    pathway mapping
- name: bioservices (Python package) v 1.7.1
  role: maps KEGG compounds to ChEBI identifiers so mapped compounds can also be matched
    against Reactome pathway sets
- name: Python 3.8
  role: runtime in which the identifier harmonisation step is executed as part of
    the simulation notebook
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: hosts reproducible_simulations.ipynb where identifier conversion is performed
    before ORA runs
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:71a8d4fea750c7e672f7ec25eb93e52f67733e3adaff245b99de8228b1f04f5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - MetaboAnalyst
  - MetaboAnalyst 4.0
  - Python
  - Python 3.8
  - bioservices (Python package) v 1.7.1
  - pip3
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# compound-identifier-mapping-to-kegg

## Summary

Converts compound names from metabolomics abundance matrices to KEGG identifiers so they can be matched against KEGG pathway sets, using the MetaboAnalyst 4.0 ID conversion tool with the bioservices package for ChEBI harmonisation.

## When to use

Use when an untargeted metabolomics dataset holds compound names that must be mapped to KEGG compound identifiers before ORA against KEGG release 96 human pathway sets.

## When NOT to use

- The dataset already uses KEGG identifiers natively — remapping through MetaboAnalyst would add conversion noise without benefit.
- The downstream pathway database is Reactome or BioCyc but only KEGG identifiers are needed — map to ChEBI/BioCyc identifiers instead of forcing KEGG IDs.
- The analysis goal is a comparison of pathway databases rather than KEGG-specific ORA — the identifier harmonisation step must span all three databases (KEGG, Reactome, BioCyc), not just KEGG.

## Inputs

- compound names from untargeted metabolomics abundance matrices (e.g. MTBLS135, MTBLS136, Yachida et al. 2019 stool, Quirós et al. 2017, Fuhrer et al. 2017 / BioStudies S-BSST5)
- KEGG release 96 human pathway compound sets
- list of differentially abundant metabolites (BH FDR q <= 0.05)

## Outputs

- KEGG compound identifiers per metabolite, harmonised with the pathway database
- mapped compounds feeding the background set assembly (nonspecific KEGG human background of 3373 compounds or assay-specific background)
- per-pathway counts k of differentially abundant metabolites for downstream Fisher's exact ORA

## How to apply

Harmonise metabolite identifiers so every compound in the dataset can be matched against the pathway database: convert compound names to KEGG identifiers with the MetaboAnalyst 4.0 ID conversion tool, and map KEGG compounds to ChEBI identifiers using the Python bioservices package (v 1.7.1) when Reactome (ChEBI) identifiers are also needed. After mapping, only pathways containing at least 3 compounds are retained as ORA input, and the mapped identifiers feed the background set assembly (e.g. the nonspecific KEGG human background of 3373 compounds versus assay-specific backgrounds). Keep the mapped KEGG IDs alongside the original compound names so that differentially abundant metabolites selected at BH FDR q <= 0.05 can be tallied as k (number of DA metabolites in a pathway), which must be >= 1 for a pathway p-value to be computed. Run the mapping as a preprocessing step inside the reproducible_simulations.ipynb Jupyter notebook (Python 3.8) so identifier conversion, background assembly, and the subsequent scipy.stats.fisher_exact ORA share a single identifier space.

## Related tools

- **MetaboAnalyst 4.0** (ID conversion tool used to convert compound names to KEGG identifiers for pathway mapping)
- **bioservices (Python package) v 1.7.1** (maps KEGG compounds to ChEBI identifiers so mapped compounds can also be matched against Reactome pathway sets)
- **Python 3.8** (runtime in which the identifier harmonisation step is executed as part of the simulation notebook) — https://github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (hosts reproducible_simulations.ipynb where identifier conversion is performed before ORA runs) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Every compound in the abundance matrix has a non-null KEGG identifier after conversion; compounds lacking a KEGG ID are documented rather than silently dropped.
- Only pathways containing at least 3 mapped compounds are used as ORA input, matching the article's filter.
- Mapped identifiers align with the background set composition (e.g. the 3373-compound nonspecific KEGG human background contains no duplicates).
- The number of differentially abundant metabolites k per pathway is derived from the mapped list, and a pathway p-value is only computed when k >= 1.
- The mapping step is reproducible inside the same Jupyter notebook run (Python 3.8) that produces pvalue_comparison.csv and sep_count_comparison.csv.

## Limitations

- Compounds that do not map to a KEGG identifier are effectively excluded from ORA, biasing the background set and potentially altering significance counts.
- The article notes that defining the ideal assay-specific background set remains an open problem, so mapping errors propagate into whichever background is chosen.
- Two of the five datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications and therefore in mapped identifiers.
- No ground-truth dataset exists where the identities of enriched pathways are known, so mapped-identifier quality cannot be directly validated against an external reference.

## Evidence

- [methods] MetaboAnalyst 4.0 ID conversion: "For the conversion of compound names to KEGG identifiers, the MetaboAnalyst 4.0 [47] ID conversion tool was used"
- [discussion] bioservices ChEBI mapping: "KEGG compounds were mapped to ChEBI identifiers using the Python bioservices package (v 1.7.1) [48]"
- [discussion] pathway filter >=3 compounds: "Only pathways containing at least 3 compounds were used as input for ORA."
- [discussion] identifier harmonisation to three databases: "In order to map compounds to the three pathway databases investigated in this study (KEGG, Reactome, and BioCyc), metabolite identifiers in each dataset were converted to the corresponding identifier"
- [discussion] nonspecific KEGG human background size: [evidence span withheld — anchor status: not_found]
- [readme] reproducible notebook execution: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
