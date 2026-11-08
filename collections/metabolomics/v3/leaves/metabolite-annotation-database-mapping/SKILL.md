---
name: metabolite-annotation-database-mapping
description: Use when METASPACE-annotated spatial or single-cell metabolomics data
  must be linked to metabolite/lipid set terms for overrepresentation or set enrichment
  analysis while accounting for isomeric and isobaric candidate ambiguity.
when_to_use_negative:
- Input is already a feature table with unambiguous, structure-resolved metabolite
  IDs (e.g. from MS/MS spectral matching) — simple ORA/MSEA without bootstrapping
  suffices.
- The MS data are MS2-based such that isomeric/isobaric molecules are already resolved
  by spectra, which MS1-only imaging data do not provide.
- No annotation database match is available for the target chemistry (e.g. non-lipid
  metabolites under a lipid-only ontology such as LION).
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: S2IsoMEr
  role: R package whose initEnrichment/Load_background functions map annotated ions
    to RAMP-DB 2.0, HMDB, SwissLipids, LipidMaps, KEGG/Reactome pathway, and LION
    metabolite-set backgrounds
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Python tool to export METASPACE annotations as AnnData for import into R as
    the mapping input
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import the exported AnnData annotation object into the S2IsoMEr
    workflow
  repo: https://github.com/scverse/anndataR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:379fff71ace5c1a3297c2e7d92715a0a4d894cbc4ea2605970c98e492a838b4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - R
  - S2IsoMEr
  - anndataR
  - dplyr
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# metabolite-annotation-database-mapping

## Summary

Maps MS1 imaging-MS annotations (METASPACE formula_adduct ions) onto curated metabolite-set databases (RAMP-DB 2.0 sub/main/super-classes, pathways from KEGG/Reactome/SMPDB/WikiPathways, HMDB, SwissLipids, LipidMaps, LION) within S2IsoMEr, so that isomeric/isobaric molecular ambiguity can be propagated into ORA/MSEA enrichment analyses.

## When to use

Use when METASPACE-annotated spatial or single-cell metabolomics data must be linked to metabolite/lipid set terms for overrepresentation or set enrichment analysis while accounting for isomeric and isobaric candidate ambiguity.

## When NOT to use

- Input is already a feature table with unambiguous, structure-resolved metabolite IDs (e.g. from MS/MS spectral matching) — simple ORA/MSEA without bootstrapping suffices.
- The MS data are MS2-based such that isomeric/isobaric molecules are already resolved by spectra, which MS1-only imaging data do not provide.
- No annotation database match is available for the target chemistry (e.g. non-lipid metabolites under a lipid-only ontology such as LION).

## Inputs

- METASPACE annotation results exported as AnnData (e.g. NASH_scm_dataset.rds from Zenodo record 13318721), filtered at FDR_level = 0.1 for DB = ('HMDB', 'v4')
- Single-cell/spatial metabolomics count matrix of detected formula_adduct ions
- Condition-to-dataset-ID mapping (reference vs query conditions)

## Outputs

- Term-to-molecule membership lists from Load_background (RAMP-DB 2.0 sub_class or pathway terms)
- Initialized S2IsoMEr enrichment object with annotated ion universe and background terms
- Per-term contingency tables feeding bootstrap ORA/MSEA (clean_enrich_res summary with intersection sizes, fold enrichment, q-values)

## How to apply

Load the annotation matrix (e.g. the SpaceM NASH single-cell metabolomics AnnData from zenodo.org/records/13318721) and restrict to detected formula_adduct ions with FDR <= 0.1 against a chosen database such as HMDB v4. Initialize the S2IsoMEr enrichment object with initEnrichment (enrichment_type = 'ORA', annot_db = 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, background_type = 'sub_class', molecule_type = 'Metabo'), which pulls term-to-molecule memberships from RAMP-DB 2.0 backgrounds; for lipids, LION, SwissLipids, and LipidMaps are supported. Each term is mapped to either molecule names or sum formulas, so enrichment can be run with or without isomeric/isobaric consideration. Use Load_background to fetch the term lists and their associated molecules, then feed the mapped term memberships into bootstrapped sampling (one candidate per annotated ion per iteration) so ambiguity propagates into term contingency tables used for Fisher's exact tests.

## Related tools

- **S2IsoMEr** (R package whose initEnrichment/Load_background functions map annotated ions to RAMP-DB 2.0, HMDB, SwissLipids, LipidMaps, KEGG/Reactome pathway, and LION metabolite-set backgrounds) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Python tool to export METASPACE annotations as AnnData for import into R as the mapping input) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import the exported AnnData annotation object into the S2IsoMEr workflow) — https://github.com/scverse/anndataR

## Examples

```
enrich_obj <- initEnrichment(scmatrix = scm_matrix, enrichment_type = 'ORA', annot_db = 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, background_type = 'sub_class', molecule_type = 'Metabo'); terms <- Load_background(enrich_obj)
```

## Evaluation signals

- All annotated ions in the input matrix are restricted to detected formula_adduct entries passing FDR <= 0.1 for the selected database (HMDB v4 in the NASH example).
- initEnrichment object reports non-empty term memberships for the chosen background_type (e.g. 'sub_class') and molecule_type ('Metabo'), sourced from RAMP-DB 2.0.
- Every mapped term carries molecule names or sum formulas, allowing enrichment with consider_isomers/consider_isobars set to TRUE.
- Downstream per-term rows show non-empty intersection sizes and bootstrap fractions >= 0.5 across >= 100 bootstrap iterations with q.val_cutoff = 0.2.
- Annotations are traceable to the METASPACE project dataset (e.g. Rappez_2021_SpaceM) used as the mapping source.

## Limitations

- MS1 reliance prevents resolving isomeric and isobaric molecules, so database mappings are ambiguous at the structure level and must be bootstrapped rather than treated as exact.
- Database coverage is limited to the supported annotation databases (CoreMetabolome, HMDB, SwissLipids, LipidMaps) and curated metabolite types from RAMP-DB 2.0; LION applies only to lipids.
- No changelog was found for the S2IsoMEr repository, so database curation versions may not be explicitly tracked.

## Evidence

- [readme] Metabolite backgrounds are curated from RAMP-DB 2.0, encompassing biological pathways and metabolic classes, which are further categorized into super, main, and sub-classes.: [evidence span withheld — anchor status: external_doc]
- [methods] The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps"): "The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps")"
- [readme] Except for the LION ontology, each term is mapped to either molecule names or sum formulas, allowing enrichment analysis to be performed with or without consideration of isomeric/isobaric ambiguity.: [evidence span withheld — anchor status: external_doc]
- [full_text] filter annotations at FDR <= 0.1 for the HMDB database, and restrict the matrix to detected formula_adduct ions: [evidence span withheld — anchor status: not_found]
- [readme] The pathways from RAMP-DB integrate multiple resources, including SMPDB, Reactome, KEGG, and WikiPathways.: [evidence span withheld — anchor status: external_doc]
