---
name: ms1-ms2-database-integration
description: Use when building or extending a plant metabolomics annotation pipeline
  that needs both MS1 exact-mass tables and MS2 spectral libraries merged into a single
  search database, e.g. for MetMiner/metID-based compound annotation.
when_to_use_negative:
- Input is already a MetMiner/metID databaseClass object or an existing PM database
  export — no recompilation is needed
- The study targets non-plant organisms; the MS1 sources (KEGG/KNApSAcK/PlantCyc)
  were filtered specifically for plant metabolites and the MS2 libraries were chosen
  for plant coverage
- Only MS1-level annotation is required and no MS2 peak lists will be matched — the
  MS2 library curation step is unnecessary overhead
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMiner
  role: Hosts the integrated PM database inside the metMiner-shinyapp and exposes
    it for metabolite annotation; provides get_compound_info() for entry standardization
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass (metID)
  role: Supplies the database-building utilities used to convert MS2 spectra and MS1
    tables into 'databaseClass' objects
- name: GNPS
  role: One of the six public MS2 spectral libraries downloaded and converted into
    the PM database
- name: KEGG
  role: Plant-filtered MS1 metabolite source (3,044 metabolites) contributing to the
    PM database
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:6f4004e517e358fed9e60e35350936b03d7dd117abb77bad78c2299bf5f805b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - GNPS
  - KEGG
  - MetMiner
  - tidyMass
  - tidyMass (metID)
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# ms1-ms2-database-integration

## Summary

Constructs a plant-specific metabolomics (PM) database by converting six MS2 spectra libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and three filtered MS1 metabolite tables (KEGG, KNApSAcK, PlantCyc) into metID 'databaseClass' objects, then embedding the combined database into the MetMiner pipeline to improve metabolite annotation accuracy and speed.

## When to use

Use when building or extending a plant metabolomics annotation pipeline that needs both MS1 exact-mass tables and MS2 spectral libraries merged into a single search database, e.g. for MetMiner/metID-based compound annotation.

## When NOT to use

- Input is already a MetMiner/metID databaseClass object or an existing PM database export — no recompilation is needed
- The study targets non-plant organisms; the MS1 sources (KEGG/KNApSAcK/PlantCyc) were filtered specifically for plant metabolites and the MS2 libraries were chosen for plant coverage
- Only MS1-level annotation is required and no MS2 peak lists will be matched — the MS2 library curation step is unnecessary overhead

## Inputs

- MS2 spectra datasets from MoNA, MassBank, ReSpect, PlaSMA, NIST and GNPS
- MS1 metabolite tables from KEGG, KNApSAcK and PlantCyc (filtered to plant entries)
- MetMiner demo files (Google Drive / Baidu Netdisk) for testing the pipeline

## Outputs

- Single combined plant-specific PM database exported as a metID 'databaseClass' structured file
- Per-source databaseClass objects (one per MS1/MS2 source)
- Summary table of metabolite counts per source (KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800)

## How to apply

Download MS2 spectral datasets from the six public libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and download plus filter the MS1 metabolite tables from the plant pathway sources, keeping the expected totals KEGG 3,044, KNApSAcK 1,627 and PlantCyc 2,800 metabolites. Standardize each entry — compound identifiers, exact masses, polarity and MS2 peak lists — using the MetMiner accessor get_compound_info() and the tidyMass metID database-building utilities. Convert each source into its own metID 'databaseClass' object, then merge the curated objects into a single plant-specific PM database and export it as a structured file that the metMiner-shinyapp loads as its annotation backend. Tabulate the per-source metabolite counts as a summary table for validation against the expected totals.

## Related tools

- **MetMiner** (Hosts the integrated PM database inside the metMiner-shinyapp and exposes it for metabolite annotation; provides get_compound_info() for entry standardization) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass (metID)** (Supplies the database-building utilities used to convert MS2 spectra and MS1 tables into 'databaseClass' objects)
- **GNPS** (One of the six public MS2 spectral libraries downloaded and converted into the PM database)
- **KEGG** (Plant-filtered MS1 metabolite source (3,044 metabolites) contributing to the PM database)

## Examples

```
library(tidymass); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Exported database file exists and loads as a valid metID 'databaseClass' object inside R/tidyMass
- Per-source count summary table reproduces the expected totals: KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800
- Each standardized entry carries compound identifier, exact mass, polarity and MS2 peak list fields populated via get_compound_info()
- metMiner-shinyapp recognizes the exported file as the PM database and uses it during the annotation step

## Limitations

- Suspected in-source fragmentation products can still be wrongly annotated as genuine metabolites; no satisfactory detection/removal solution exists and this is planned for the next MetMiner version
- No release DOI, version number, or persistence guarantee is stated for the pipeline's hosted components, complicating exact reproduction of the database build
- MS1 sources must be filtered to plant entries before conversion; unfiltered KEGG/PlantCyc dumps will not match the expected per-source totals

## Evidence

- [abstract] PM database built from six MS2 + three MS1 sources: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] Design rationale for integrating MS1 and MS2 databases: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [discussion] PM database purpose and integration point: [evidence span withheld — anchor status: not_found]
- [intro] metID/tidyMass provides the database-building utilities: "The upstream data processing (peak picking and data cleaning) is driven by tidyMass, inheriting the advantages of reproducibility, traceability, and transparency."
- [readme] Launching the shinyapp that consumes the PM database: "library(tidyverse)
library(tidymass)
library(MDAtoolkits)
library(MetMiner)
##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
