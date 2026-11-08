---
name: pathway-activity-scoring
description: Use when a peak-feature intensity matrix (samples as columns, group row)
  plus a peak-to-compound annotation table is available and pathway-level ranking
  of significantly changing metabolite sets is the analysis goal.
when_to_use_negative:
- Many peak features lack compound annotations — unannotated peaks are simply not
  used for pathway analysis, degrading coverage
- Goal is all Reactome pathways in offline mode — offline mode provides only metabolic
  pathways; full-pathway or up-to-date queries need an online Neo4j Reactome instance
- Input is raw spectra needing feature detection first — PALS expects a processed
  peak-intensity matrix, not centroided/profile MS data
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0602
tools:
- name: PALS
  role: Performs the pathway activity level scoring itself — database pathway queries,
    PLAGE decomposition, and ORA/GSEA comparison runs; installable via `pip install
    PALS-pathway`
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds the intensity matrix (int_df) and annotation frame (annotation_df) passed
    directly into pals.DataSource for programmatic use
- name: SciPy
  role: Standardises the imputed, log2-transformed intensity matrix (zero mean, unit
    variance) via its preprocessing module
- name: PALS Viewer
  role: Streamlit web interface for running PALS, ranking pathways, and prioritising
    GNPS Molecular Families / MS2LDA Mass2Motifs by activity level
  repo: https://github.com/glasgowcompbio/PALS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:efaeee0a724b9fdf230f143761330ae7f6a27a7da610b4bb6c145f0cc6a9d61d
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
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9791666666666666
n_claims: 48
---

# pathway-activity-scoring

## Summary

Rank metabolite sets (KEGG/Reactome pathways, GNPS Molecular Families, MS2LDA Mass2Motifs) by pathway activity level using the PLAGE decomposition in the PALS library, which is more robust to missing peaks and noise than overrepresentation analysis (ORA) or GSEA.

## When to use

Use when a peak-feature intensity matrix (samples as columns, group row) plus a peak-to-compound annotation table is available and pathway-level ranking of significantly changing metabolite sets is the analysis goal.

## When NOT to use

- Many peak features lack compound annotations — unannotated peaks are simply not used for pathway analysis, degrading coverage
- Goal is all Reactome pathways in offline mode — offline mode provides only metabolic pathways; full-pathway or up-to-date queries need an online Neo4j Reactome instance
- Input is raw spectra needing feature detection first — PALS expects a processed peak-intensity matrix, not centroided/profile MS data

## Inputs

- Intensity CSV: peak features as rows, samples as columns, with a second row assigning samples to experimental groups (e.g. group,beer1,beer1,...)
- Annotation CSV/matrix: two columns mapping peak ID (row_id) to metabolite annotation as KEGG (e.g. C00148) or ChEBI IDs, one-to-many allowed
- Experimental design dictionary (programmatic use): 'groups' and 'comparisons' entries naming case/control pairs
- Pathway database selection: PiMP_KEGG, COMPOUND, ChEBI, UniProt, or ENSEMBL

## Outputs

- Pathway ranking file (CSV) with pathway id, p-value columns, unq_pw_F (unique formulae in pathway), tot_ds_F (formula hits in dataset), and F_coverage columns
- Per-method pathway p-value DataFrame via method.get_pathway_df() when used as a Python library

## How to apply

Load the intensity matrix and annotation file (e.g. notebooks/test_data/HAT/int_df.csv and annotation_df.csv) as pandas DataFrames; peaks without annotations are excluded from pathway analysis. On loading, PALS imputes zero intensities: all-zero factors are replaced by --min_replace (default 5000), partially-zero factors by the mean of non-zero samples in that factor; data are then log-2 transformed and standardised with SciPy's preprocessing module to zero mean and unit variance. Choose the pathway database (PiMP_KEGG, COMPOUND, ChEBI for metabolomics; UniProt/ENSEMBL for proteomics/transcriptomics) and specify comparisons as case/control pairs, e.g. Stage_1/Control Stage_2/Control. Run pathway ranking via pals.PLAGE (mPLAGE), pals.ORA, or pals.GSEA using the DataSource API, or via the run_pals.py CLI; benchmarking in the paper shows mPLAGE yields lower mean p-values with smaller variance and better robustness as missing features increase from 20% to 80%. Sort the output pathway ranking file by its p-value columns and inspect the unq_pw_F, tot_ds_F, and F_coverage columns (proportion of dataset formula hits to pathway formulae) to gauge pathway coverage.

## Related tools

- **PALS** (Performs the pathway activity level scoring itself — database pathway queries, PLAGE decomposition, and ORA/GSEA comparison runs; installable via `pip install PALS-pathway`) — https://github.com/glasgowcompbio/PALS
- **pandas** (Holds the intensity matrix (int_df) and annotation frame (annotation_df) passed directly into pals.DataSource for programmatic use)
- **SciPy** (Standardises the imputed, log2-transformed intensity matrix (zero mean, unit variance) via its preprocessing module)
- **PALS Viewer** (Streamlit web interface for running PALS, ranking pathways, and prioritising GNPS Molecular Families / MS2LDA Mass2Motifs by activity level) — https://github.com/glasgowcompbio/PALS

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control
```

## Evaluation signals

- Output ranking file contains the documented coverage columns (unq_pw_F, tot_ds_F, F_coverage) and can be sorted by p-value columns
- In robustness benchmarks (20–80% of features randomly removed, 500 seeds, decoy level fixed at 100%), mPLAGE yields consistently lower mean p-values with smaller p-value variance than ORA and GSEA, with the best F1 scores
- Zero-imputation behaves as specified: all-zero factors replaced by the min_replace value (default 5000), partially-zero factors by the mean of non-zero samples, followed by log2 transform and zero-mean/unit-variance standardisation
- Columns of int_df map correctly onto the experimental design comparisons (each case/control pair appears in the output)

## Limitations

- No software version numbers stated in the article; for updates one must check the repository, and no changelog was found
- Offline Reactome mode only exposes metabolic pathways; querying all or updated pathways requires --connect_to_reactome_server and a local Neo4j instance
- Multiple peak IDs may map to multiple compound IDs and vice versa due to identification uncertainty, which propagates ambiguity into pathway membership

## Evidence

- [abstract] we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets: "we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets"
- [methods] if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [methods] standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA).: "The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA)."
- [readme] The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset: "Pathways are identified by their id and can be sorted by the `p-value` columns. The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the"
