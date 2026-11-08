---
name: pathway-activity-level-scoring
description: Use when you have a peak-intensity matrix plus peak-to-compound annotations
  (KEGG/ChEBI IDs) or fragmentation-based metabolite sets (GNPS Molecular Families,
  Mass2Motifs) and need to rank metabolite sets by activity between a case and control
  group.
when_to_use_negative:
- Input is already a ranked or normalised pathway-level table rather than raw peak
  intensities needing decomposition
- Peak features carry no compound annotations and no fragmentation-based groupings
  (Molecular Families/Mass2Motifs) exist — unannotated peaks are not used for pathway
  analysis
- 'The goal is transcriptomics/proteomics pathway queries without the matching database:
  UniProt/ENSEMBL Reactome databases, not the metabolomics PiMP_KEGG/ChEBI options,
  must be selected'
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: PALS
  role: Core library/CLI that performs imputation, log-2 transformation, standardisation,
    and pathway ranking via PLAGE, ORA, or GSEA
  repo: https://github.com/glasgowcompbio/PALS
- name: PALS Viewer
  role: Streamlit web interface for running PALS and prioritising Molecular Families
    and Mass2Motifs by activity levels
  repo: https://pals.glasgowcompbio.org/app/
- name: pandas
  role: Passes intensity and annotation dataframes directly to PALS when used programmatically
- name: SciPy
  role: Preprocessing module used to standardise the intensity matrix to zero mean
    and unit variance
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:1cd7aba4a0119ed6c6590006c4edb5efad94a583c9753ef21f8244b21ad59a5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - PALS Viewer
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# pathway-activity-level-scoring

## Summary

Rank significantly changing metabolite sets (KEGG pathways, GNPS Molecular Families, or MS2LDA Mass2Motifs) using PALS, which decomposes pathway activity levels via the PLAGE method and is more robust to missing peaks and noise than ORA or GSEA.

## When to use

Use when you have a peak-intensity matrix plus peak-to-compound annotations (KEGG/ChEBI IDs) or fragmentation-based metabolite sets (GNPS Molecular Families, Mass2Motifs) and need to rank metabolite sets by activity between a case and control group.

## When NOT to use

- Input is already a ranked or normalised pathway-level table rather than raw peak intensities needing decomposition
- Peak features carry no compound annotations and no fragmentation-based groupings (Molecular Families/Mass2Motifs) exist — unannotated peaks are not used for pathway analysis
- The goal is transcriptomics/proteomics pathway queries without the matching database: UniProt/ENSEMBL Reactome databases, not the metabolomics PiMP_KEGG/ChEBI options, must be selected

## Inputs

- Peak-intensity CSV matrix (rows = peak features, columns = samples, with an optional second 'group' line mapping samples to experimental factors)
- Annotation CSV mapping peak IDs to KEGG or ChEBI compound IDs (two columns: row_id, entity_id; many-to-many mappings allowed)
- experimental_design dictionary with 'groups' and 'comparisons' (case/control) when using PALS programmatically
- User-defined metabolite sets (GNPS Molecular Families, MS2LDA Mass2Motifs) for non-pathway analyses
- min_replace imputation value (default 5000)

## Outputs

- Pathway ranking table sorted by p-value with columns p-value, unq_pw_F (unique formulae in pathway), tot_ds_F (formula hits in dataset), and F_coverage (tot_ds_F / unq_pw_F)
- Pathway activity levels (PLAGE decomposition) per metabolite set per comparison

## How to apply

Provide PALS with a peak-intensity CSV (rows = peak features, second line mapping samples to experimental groups) and an annotation CSV mapping peak IDs to KEGG or ChEBI compound IDs; peaks without annotations are excluded from pathway analysis. On loading, PALS imputes zero intensities within each experimental factor using min_replace (default 5000) when all samples in a factor are zero, or the mean of non-zero samples when only some are zero, then log-2 transforms and standardises the matrix to zero mean and unit variance with the SciPy preprocessing module. Choose the pathway database (PiMP_KEGG, COMPOUND, ChEBI for metabolomics; UniProt, ENSEMBL for proteomics/transcriptomics), a ranking method (PLAGE, ORA, or GSEA), and the case/control comparison (e.g. --comparisons Stage_1/Control). For unknown metabolites, build a DataSource with user-defined metabolite sets from GNPS Molecular Families or MS2LDA Mass2Motifs and run PLAGE(ds).get_pathway_df() programmatically, passing an experimental_design dict of groups and comparisons. Output pathways are ranked by p-value with unq_pw_F, tot_ds_F, and F_coverage columns; prefer mPLAGE over ORA/GSEA when annotation completeness is poor, since the benchmark showed mPLAGE returns lower median p-values (e.g. 0.0049 vs ORA 2e-6 at 0% decoy but more robust at high decoy levels, where GSEA's median p-value was 0.0578 at 0% decoy).

## Related tools

- **PALS** (Core library/CLI that performs imputation, log-2 transformation, standardisation, and pathway ranking via PLAGE, ORA, or GSEA) — https://github.com/glasgowcompbio/PALS
- **PALS Viewer** (Streamlit web interface for running PALS and prioritising Molecular Families and Mass2Motifs by activity levels) — https://pals.glasgowcompbio.org/app/
- **pandas** (Passes intensity and annotation dataframes directly to PALS when used programmatically)
- **SciPy** (Preprocessing module used to standardise the intensity matrix to zero mean and unit variance)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Output CSV contains the p-value, unq_pw_F, tot_ds_F, and F_coverage columns and can be sorted by p-value
- F_coverage equals tot_ds_F divided by unq_pw_F for every ranked pathway
- Reproducing the synthetic benchmark: at 0% decoy features ORA and mPLAGE return median p-values of 2e-6 and 0.0049, and GSEA 0.0578, over 500 repetitions within Monte Carlo tolerance
- mPLAGE yields lower median p-values than ORA and GSEA at high (250-1000%) decoy-feature levels, confirming robustness to missing annotations
- Known biology recovered: flavonoid-related Mass2Motifs (rhamnocitrin, kaempferol, flavonoid core, emodin) score higher in Rhamnus while the cyclopeptidic-alkaloids motif scores higher in Ziziphus

## Limitations

- Peaks without compound annotations will not be used for pathway analysis, so annotation completeness directly degrades results (hence the decoy-feature benchmark)
- No software version numbers or changelog are stated in the article; the development version must be checked out from Git
- In offline Reactome mode, only metabolic pathways are available; full pathway sets require online Neo4j mode (--use_all_reactome_pathways --connect_to_reactome_server)

## Evidence

- [abstract] PALS performs database queries of pathways, decomposes activity levels via PLAGE, and is robust to missing peaks: "we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets"
- [readme] Imputation rule and transformation pipeline: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] Standardisation via SciPy: "transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] Output columns definition: "The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset, and `F_coverage` is the proportion"
- [abstract] mPLAGE robustness finding: "reveals that mPLAGE is more robust to missing"
- [readme] Fragmentation-based metabolite sets supported: "This includes in particular *Molecular Families* from [GNPS](http://gnps.ucsd.edu/), as well as *Mass2Motifs* from [MS2LDA](http://ms2lda.org/)."
- [readme] Unannotated peaks excluded: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
