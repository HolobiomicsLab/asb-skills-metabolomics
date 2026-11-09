---
name: mass-spectral-library-construction
description: Use when constructing a plant-specific annotation reference from public
  MS2 spectra and MS1 metabolite tables to feed a tidyMass/metID-based annotation
  workflow, and per-source metabolite counts must be tracked and validated.
when_to_use_negative:
- 'Your sample matrix is non-plant (e.g. human/metagenomic): use organism-appropriate
  references such as the Human Metabolome Database instead of the plant-specific PM
  database'
- Your annotation pipeline is not tidyMass/metID-based and cannot load 'databaseClass'
  objects
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: MetMiner
  role: Pipeline into which the plant-specific MS database is integrated; get_compound_info()
    standardizes compound identifiers, exact masses, polarity and MS2 peak lists;
    metMiner-shinyapp hosts the PM database
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Supplies the metID database-building utilities used to convert curated MS2
    spectra and MS1 tables into 'databaseClass' objects
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- name: GNPS
  role: One of the six public MS2 spectral libraries downloaded as a source for the
    PM database
- name: KEGG
  role: Plant-specific MS1 pathway source contributing 3,044 metabolites to the PM
    database
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:54e8af35769f42d3f9ef696c7fef16ac613c8234f6c972c3ffb66e64c93f6f8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - GNPS
  - KEGG
  - MetMiner
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# mass-spectral-library-construction

## Summary

Build a plant-specific mass spectrometry (PM) database by converting public MS2 spectral libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and filtered MS1 tables (KEGG, KNApSAcK, PlantCyc) into metID 'databaseClass' objects for embedding into the MetMiner pipeline to improve metabolite annotation.

## When to use

Use when constructing a plant-specific annotation reference from public MS2 spectra and MS1 metabolite tables to feed a tidyMass/metID-based annotation workflow, and per-source metabolite counts must be tracked and validated.

## When NOT to use

- Your sample matrix is non-plant (e.g. human/metagenomic): use organism-appropriate references such as the Human Metabolome Database instead of the plant-specific PM database
- Your annotation pipeline is not tidyMass/metID-based and cannot load 'databaseClass' objects

## Inputs

- MS2 spectral datasets from MoNA, MassBank, ReSpect, PlaSMA, NIST and GNPS
- MS1 metabolite tables from KEGG (3,044 metabolites), KNApSAcK (1,627) and PlantCyc (2,800)
- Compound identifiers, exact masses, polarity and MS2 peak lists per entry

## Outputs

- One metID 'databaseClass' object per source database
- A single combined plant-specific PM database exported as a structured file, embedded in the metMiner-shinyapp
- A summary table of metabolite counts contributed by each source

## How to apply

Download MS2 spectral datasets from the six public libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and MS1 metabolite tables from the plant-specific pathway sources, filtering to keep KEGG (3,044 metabolites), KNApSAcK (1,627) and PlantCyc (2,800). Standardize each entry — compound identifiers, exact masses, polarity, and MS2 peak lists — using MetMiner's get_compound_info() accessor and the metID database-building utilities from tidyMass. Convert each curated source into its own metID 'databaseClass' object, then merge the MS2 spectra and MS1 tables into a single plant-specific PM database and export it as a structured file. Tabulate the number of metabolites contributed by each source and embed the PM database into the metMiner-shinyapp so annotation accuracy and speed are optimized for plant metabolomics.

## Related tools

- **MetMiner** (Pipeline into which the plant-specific MS database is integrated; get_compound_info() standardizes compound identifiers, exact masses, polarity and MS2 peak lists; metMiner-shinyapp hosts the PM database) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Supplies the metID database-building utilities used to convert curated MS2 spectra and MS1 tables into 'databaseClass' objects) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- **GNPS** (One of the six public MS2 spectral libraries downloaded as a source for the PM database)
- **KEGG** (Plant-specific MS1 pathway source contributing 3,044 metabolites to the PM database)

## Examples

```
if (!require('tidymass')) { source("https://www.tidymass.org/tidymass-packages/install_tidymass.txt"); install_tidymass(from = "tidymass.org") }; remotes::install_github('ShawnWx2019/MetMiner')
```

## Evaluation signals

- The exported database file exists and loads as a metID 'databaseClass' object in R
- The per-source count table reproduces the expected totals: KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800
- The PM database is loadable in the metMiner-shinyapp and is used for metabolite annotation with improved accuracy/speed
- Each per-source 'databaseClass' object carries standardized compound identifiers, exact masses, polarity and MS2 peak lists

## Limitations

- Some suspected in-source fragmentation products are annotated as genuine metabolites and there is no satisfactory solution to detect and remove such redundant annotations; addressing this is a major focus for the next version of MetMiner
- MetMiner is lifecycle 'Experimental' (README badge) with no version number stated in the article for MetMiner, tidyMass, or MDAtoolkits

## Evidence

- [abstract] we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] Plant-specific databases containing MS1 and MS2 spectra are integrated to enhance metabolite annotation accuracy: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [discussion] we also integrated a plant-specific metabolite database in MetMiner, to enhance the accuracy and speed for metabolite annotation in plant metabolomics: [evidence span withheld — anchor status: not_found]
- [discussion] Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner: "Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner."
- [readme] lifecycle badge Experimental; if (!require('tidymass')) install_tidymass(from = "tidymass.org"): "![lifecycle](https://img.shields.io/badge/lifecycle-Experimental-lightcyan)"
