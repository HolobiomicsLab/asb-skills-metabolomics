---
name: metid-databaseclass-object-construction
description: Use when you need to compile or extend a plant-specific MS1/MS2 annotation
  database from public spectral libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST,
  GNPS) and MS1 sources (KEGG, KNApSAcK, PlantCyc) for use in MetMiner/tidyMass.
when_to_use_negative:
- Input is raw LC-MS vendor data needing peak picking or data cleaning — that is the
  tidyMass-driven upstream processing stage of MetMiner, not database construction
- A ready-made metID databaseClass object (e.g. the embedded PM database) already
  exists and only needs to be used for annotation
- 'The target organisms are non-plant: the PM database is plant-specific, and human/general
  resources such as HMDB or MetFlow contexts call for different MS1 sources'
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: MetMiner
  role: Pipeline into which the constructed PM database is embedded (metMiner-shinyapp);
    supplies get_compound_info() for standardizing entries
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Provides the metID package and database-building utilities that create the
    databaseClass objects
- name: KEGG
  role: MS1 metabolite source contributing 3,044 plant metabolites; accessed via get_kegg_pathway_ont()
- name: GNPS
  role: One of the six public MS2 spectral libraries downloaded for the PM database
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:934d78d4a6c248373c7596946c69f95e8cc7262950e2eb95254a535eed44f7ca
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
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metid-databaseclass-object-construction

## Summary

Builds a plant-specific metabolite database (PM database) by converting six MS2 spectra libraries and three filtered MS1 metabolite tables into metID 'databaseClass' objects for embedding in the metMiner-shinyapp annotation pipeline.

## When to use

Use when you need to compile or extend a plant-specific MS1/MS2 annotation database from public spectral libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and MS1 sources (KEGG, KNApSAcK, PlantCyc) for use in MetMiner/tidyMass.

## When NOT to use

- Input is raw LC-MS vendor data needing peak picking or data cleaning — that is the tidyMass-driven upstream processing stage of MetMiner, not database construction
- A ready-made metID databaseClass object (e.g. the embedded PM database) already exists and only needs to be used for annotation
- The target organisms are non-plant: the PM database is plant-specific, and human/general resources such as HMDB or MetFlow contexts call for different MS1 sources

## Inputs

- MS2 spectral datasets downloaded from MoNA, MassBank, ReSpect, PlaSMA, NIST and GNPS
- MS1 metabolite tables from plant-specific pathway sources KEGG, KNApSAcK and PlantCyc
- Per-compound fields: compound identifiers, exact masses, polarity, and MS2 peak lists

## Outputs

- One metID 'databaseClass' object per source (MS2 and MS1)
- A single combined plant-specific PM database exported as a structured file
- A summary table of metabolite counts per source (KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800)

## How to apply

Download the MS2 spectral datasets from the six public libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and download plus filter the MS1 metabolite tables from KEGG (3,044 metabolites), KNApSAcK (1,627) and PlantCyc (2,800). Standardize every entry — compound identifiers, exact masses, polarity and MS2 peak lists — using MetMiner's get_compound_info() accessor and the metID database-building utilities from tidyMass, so each source conforms to the databaseClass schema. Merge each curated source into its own metID databaseClass object (one per source), then combine them into the single plant-specific PM database and export it as a structured file. Tabulate the number of metabolites contributed by each source as a summary table, and embed the finished PM database into the metMiner-shinyapp so it drives metabolite annotation accuracy and speed. Validation rests on the exported file loading as a databaseClass object and the per-source counts reproducing the expected totals (KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800).

## Related tools

- **MetMiner** (Pipeline into which the constructed PM database is embedded (metMiner-shinyapp); supplies get_compound_info() for standardizing entries) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Provides the metID package and database-building utilities that create the databaseClass objects)
- **KEGG** (MS1 metabolite source contributing 3,044 plant metabolites; accessed via get_kegg_pathway_ont())
- **GNPS** (One of the six public MS2 spectral libraries downloaded for the PM database)

## Examples

```
source("https://www.tidymass.org/tidymass-packages/install_tidymass.txt"); install_tidymass(from = "tidymass.org")
```

## Evaluation signals

- The exported database file exists and loads successfully as a metID 'databaseClass' object
- The per-source count table reproduces the expected totals: KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800
- Every database entry carries standardized compound identifiers, exact masses, polarity and MS2 peak lists
- The combined PM database is loadable within the metMiner-shinyapp and labeled as the PM database
- MS2 and MS1 components are both present, matching the paper's design of integrating MS1 and MS2 spectra for annotation

## Limitations

- No version number, release DOI or containerization is documented for MetMiner, tidyMass or MDAtoolkits, so exact package versions used for database construction are unrecorded
- Currently there is no satisfactory solution to detect and remove in-source fragmentation products wrongly annotated as genuine metabolites — a known annotation limitation the database inherits
- Coverage is bounded by the chosen sources (six MS2 libraries plus KEGG/KNApSAcK/PlantCyc); metabolites outside these resources will not be annotated

## Evidence

- [abstract] PM database integrated into MetMiner: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] MS1 and MS2 integration rationale: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [discussion] PM database purpose: [evidence span withheld — anchor status: not_found]
- [readme] tidyMass/metID dependency: "source("https://www.tidymass.org/tidymass-packages/install_tidymass.txt"); install_tidymass(from = "tidymass.org")"
- [methods] KEGG access method: "get_kegg_pathway_ont(ont)"
