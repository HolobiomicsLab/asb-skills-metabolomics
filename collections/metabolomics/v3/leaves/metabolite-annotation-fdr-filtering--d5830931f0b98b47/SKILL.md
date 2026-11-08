---
name: metabolite-annotation-fdr-filtering
description: Use when building marker/query and universe sets for metabolite enrichment
  from METASPACE annotation tables, and an FDR-controlled significance cutoff (e.g.
  0.1) must be applied per annotation database before ORA or MSEA.
when_to_use_negative:
- Input is already a curated list of confidently resolved metabolite structures (e.g.
  MS2-confirmed IDs) where isomeric/isobaric ambiguity propagation is unnecessary
- The annotation source is not METASPACE and does not provide per-annotation FDR estimates
- The goal is annotation quality control or calibration of the FDR itself rather than
  selecting significant annotations for downstream enrichment
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: METASPACE
  role: Source of FDR-controlled formula-adduct annotation results for imaging MS
    datasets, queried per dataset and database
  repo: https://metaspace2020.org/
- name: S2IsoMEr
  role: Consumes the FDR-passing annotation query and detected-annotation universe
    as the custom_universe for bootstrapping ORA/MSEA
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Downloads and converts METASPACE datasets to AnnData for import into R prior
    to annotation filtering
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: Imports the AnnData-converted METASPACE dataset into R so annotations can
    be filtered against matrix rownames
  repo: https://github.com/scverse/anndataR
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:dd68dfc01b80403267252dad132dcfc826712ed948228b68496a198855e96aac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - anndataR
  - dplyr
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-annotation-fdr-filtering

## Summary

Filter FDR-controlled METASPACE formula-adduct annotation results (e.g. at FDR <= 0.1 against HMDB v4 or CoreMetabolome) so that only passing annotations form the ORA/MSEA query set, while all detected annotations serve as the custom universe for S2IsoMEr bootstrapping enrichment.

## When to use

Use when building marker/query and universe sets for metabolite enrichment from METASPACE annotation tables, and an FDR-controlled significance cutoff (e.g. 0.1) must be applied per annotation database before ORA or MSEA.

## When NOT to use

- Input is already a curated list of confidently resolved metabolite structures (e.g. MS2-confirmed IDs) where isomeric/isobaric ambiguity propagation is unnecessary
- The annotation source is not METASPACE and does not provide per-annotation FDR estimates
- The goal is annotation quality control or calibration of the FDR itself rather than selecting significant annotations for downstream enrichment

## Inputs

- METASPACE annotation results table with FDR values, formula-adduct identifiers, and annotation database (e.g. SpaceM_metaspace_ds_annotations.rds from Zenodo record 13318721)
- Annotation database name and version (e.g. HMDB v4, CoreMetabolome, SwissLipids, LipidMaps)
- FDR cutoff level (e.g. 0.1)
- Condition labels defining the comparison (e.g. U vs F in the HepaRG NASH dataset)
- Single-cell or spatial metabolomics matrix whose rownames intersect the annotations (e.g. NASH_scm_dataset.rds)

## Outputs

- FDR-passing annotation set used as the ORA/MSEA marker query (formula-adduct identifiers)
- Custom universe of all detected annotations for the selected database
- Condition-filtered annotation-to-matrix mapping ready for S2IsoMEr initEnrichment

## How to apply

Retrieve FDR-controlled formula-adduct identifiers from METASPACE for the dataset of interest (e.g. 2022-05-31_10h46m34s at FDR 10% against CoreMetabolome, or SpaceM NASH annotations against HMDB v4), selecting the annotation database and FDR level explicitly (FDR_level = 0.1, DB = HMDB v4). Subset the annotation table to rows passing the FDR cutoff for the conditions being compared (e.g. 'U' vs 'F' in the HepaRG steatosis model), and intersect the passing formula_adduct ions with the metabolite matrix rownames to define the ORA query. Use the full set of detected annotations (FDR-passing or not) as the custom universe so enrichment is computed against what was actually measured. Pass these sets into S2IsoMEr (initEnrichment with enrichment_type='ORA', annot_db='HMDB', consider_isomers=TRUE, consider_isobars=TRUE) because MS1-only imaging MS cannot resolve isomeric and isobaric identifications, so downstream enrichment must propagate that ambiguity via bootstrapping rather than treating FDR-filtered IDs as unambiguous.

## Related tools

- **METASPACE** (Source of FDR-controlled formula-adduct annotation results for imaging MS datasets, queried per dataset and database) — https://metaspace2020.org/
- **S2IsoMEr** (Consumes the FDR-passing annotation query and detected-annotation universe as the custom_universe for bootstrapping ORA/MSEA) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Downloads and converts METASPACE datasets to AnnData for import into R prior to annotation filtering) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (Imports the AnnData-converted METASPACE dataset into R so annotations can be filtered against matrix rownames) — https://github.com/scverse/anndataR

## Examples

```
query <- dplyr::filter(annot, FDR <= 0.1, database == "HMDB", condition %in% c("U","F"))
```

## Evaluation signals

- Every identifier in the query set has FDR <= 0.1 for the selected database (e.g. HMDB v4), and no annotations from other databases leak in
- The query set is a strict subset of the custom universe of all detected annotations, and query identifiers intersect the metabolite matrix rownames
- Downstream ORA output (metaspace_ORA_enrichment_results.csv) exists and contains Term/q-value fields with terms passing q.val_cutoff = 0.2 and boot_fract_cutoff = 0.5 in passed_filters_per_term
- Expected positive control terms appear after filtering: 'phosphosphingolipids'/'sphingolipids' for the mouse brain LION analysis, or 'triacylglycerols'/'diradylglycerols' for steatotic HepaRG cells
- Condition filtering retains only cells/pixels labeled with the compared conditions (e.g. 'U' and 'F') before intersecting with annotations

## Limitations

- Reliance on MS1 data in imaging MS prevents resolving isomeric and isobaric molecules, so an FDR filter alone cannot guarantee structurally unambiguous annotations — ambiguity must be propagated by bootstrapping (S2IsoMEr)
- FDR <= 0.1 still admits a nominal 10% expected false discovery rate among kept formula-adduct IDs; ambiguous annotations can inflate term significance in standard ORA (terms significant only under Run_simple_ORA, with ambiguity scores < 0.5, are likely false positives)
- Filtering is database-specific: results at FDR 0.1 against CoreMetabolome are not interchangeable with HMDB v4 or LION-based backgrounds

## Evidence

- [abstract] Reliance on MS1 blocks resolving isomers: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [methods] FDR level and database selection: "FDR_level = 0.1
DB = ("HMDB", "v4")"
- [methods] Supported annotation databases for filtering: "The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps")"
- [methods] Retrieving METASPACE datasets: "use the [METASPACE converter](https://github.com/metaspace2020/metaspace-converter) to transform [METASPACE](https://metaspace2020.org/) datasets"
- [readme] S2IsoMEr addresses identification ambiguity in imaging MS: [evidence span withheld — anchor status: external_doc]
