---
name: tissue-distribution-visualization
description: Use when you hold MASST match results for synthesized compounds (e.g.,
  conjugated bile acids) annotated with ReDU biospecimen/tissue and health-phenotype
  metadata, and you need tissue-distribution or disease-association count tables and
  ordinations.
when_to_use_negative:
- You need peak-level or per-sample metabolite quantification rather than aggregate
  spectral-match counts across public datasets
- The matched public spectra lack ReDU metadata (no biospecimen/tissue or health-phenotype
  annotation), so grouping and visualization would be meaningless
- Your compounds are not represented by MASST-searched spectra (e.g., metabolites
  absent from the ~1.2 billion public MASST-searched spectra)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0092
tools:
- name: MASST
  role: Source of the matched public spectra searched with synthesized conjugated
    bile acid MS/MS spectra; the match counts and their ReDU metadata are what get
    aggregated and visualized
- name: Nextflow
  role: Workflow framework in the Reverse_metabolomics_library_generation repo used
    to run the library-generation/MASST-search pipeline whose outputs feed the tissue-distribution
    tables
  repo: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:f5fff75ca5cb69bc8bc56f4fb5c2abb82f2511b06033c220848142266acbe7ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation
  tools:
  - MASST
  - Nextflow
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# tissue-distribution-visualization

## Summary

Aggregates MASST spectral matches to synthesized conjugated bile acids by ReDU sample metadata (biospecimen/tissue and health phenotype) and renders count tables/ordinations (Fig. 2c,d) that reveal where bile amidates occur in the human body and how they track IBD status.

## When to use

Use when you hold MASST match results for synthesized compounds (e.g., conjugated bile acids) annotated with ReDU biospecimen/tissue and health-phenotype metadata, and you need tissue-distribution or disease-association count tables and ordinations.

## When NOT to use

- You need peak-level or per-sample metabolite quantification rather than aggregate spectral-match counts across public datasets
- The matched public spectra lack ReDU metadata (no biospecimen/tissue or health-phenotype annotation), so grouping and visualization would be meaningless
- Your compounds are not represented by MASST-searched spectra (e.g., metabolites absent from the ~1.2 billion public MASST-searched spectra)

## Inputs

- Filtered MASST match results table with matched public spectra for synthesized conjugated bile acids
- ReDU sample metadata per match: biospecimen/tissue category and health phenotype (IBD vs healthy)
- Mapping of each MASST match to its bile acid conjugate type (e.g., cholic acid conjugated to Glu, Ile/Leu, Phe, Thr, Trp, Tyr)

## Outputs

- Tissue-by-conjugate spectral count table (Fig. 2c style) showing concentration of matches in fecal material and intestinal tissue
- Phenotype-by-conjugate spectral count table (Fig. 2d style) comparing IBD vs healthy match frequencies
- count_table.csv containing the aggregated tissue and phenotype count tables
- Jaccard-distance ordination of samples/groups colored by health status

## How to apply

Load the filtered MASST match results with each matched public spectrum plus its attached ReDU metadata (biospecimen/tissue, health phenotype such as IBD vs healthy). Group matched spectral counts by ReDU biospecimen/tissue category and bile acid conjugate type (Glu, Ile/Leu, Phe, Thr, Trp, Tyr, etc.) to tabulate counts per tissue-conjugate pair (Fig. 2c), and separately by health phenotype to tabulate counts per phenotype-conjugate pair (Fig. 2d). Compute Jaccard distances between groups based on shared bile acid conjugates and run ordination on the health-status grouping to check whether samples cluster by health status. Rationale: repository-scale MASST searching of ~2,000 synthesized compounds against public metabolomics data (e.g., iHMP2 stool on Thermo QE Orbitrap in negative mode) only becomes interpretable when matches are aggregated over ReDU metadata, turning per-spectrum hits into phenotype-level distribution patterns. Write the tissue-by-conjugate and phenotype-by-conjugate tables to count_table.csv and inspect that conjugated bile acids appear mainly in fecal material and intestinal tissue, rarely in blood/liver.

## Related tools

- **MASST** (Source of the matched public spectra searched with synthesized conjugated bile acid MS/MS spectra; the match counts and their ReDU metadata are what get aggregated and visualized)
- **Nextflow** (Workflow framework in the Reverse_metabolomics_library_generation repo used to run the library-generation/MASST-search pipeline whose outputs feed the tissue-distribution tables) — https://github.com/Wang-Bioinformatics-Lab/Reverse_metabolomics_library_generation

## Examples

```
import pandas as pd; m = pd.read_csv('masst_matches.tsv', sep='\t'); m.groupby(['biospecimen','conjugate']).size().to_csv('count_table.csv')
```

## Evaluation signals

- The tissue-by-conjugate table reproduces the reported pattern: conjugated bile acids detected mainly in fecal material and intestinal tissue, rarely in blood/liver
- The phenotype-by-conjugate table reproduces increased counts for cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr in Crohn's disease/IBD samples relative to healthy (Fig. 2c,d, validated across four distinct human IBD cohorts)
- Jaccard-distance ordination shows samples clustering by health status grouping
- Every spectral count in count_table.csv is traceable to a MASST match with intact ReDU biospecimen/phenotype metadata (no matches dropped silently)

## Limitations

- The 1.2 billion spectra currently part of MASST searches do not encompass all molecules, limiting re-analyses of public data in the discovery phase
- Without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded, so conjugate-level annotations may conflate isomers
- The protocol does not guarantee against false discovery of closely related isomers or compounds, though it prevents completely inaccurate identifications
- Sparse metadata in public depositions limits the tissue/phenotype grouping; increased public data deposition with metadata is needed

## Evidence

- [abstract] Repository-scale IBD association finding: "Using repository-scale analysis 1,2 , we discovered that some conjugated bile acids are associated with inflammatory bowel disease (IBD)."
- [supplementary] iHMP2 stool detection context: "conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode."
- [discussion] MASST public-data limitation: "Last, the 1.2 billion spectra that are currently part of MASST searches do not encompass all molecules."
- [abstract] Reverse metabolomics definition: "whereby tandem mass spectrometry spectra acquired from newly synthesized compounds are searched for in public metabolomics datasets to uncover phenotypic associations"
