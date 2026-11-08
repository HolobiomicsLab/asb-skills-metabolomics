---
name: plant-metabolite-source-curation
description: 'Use when building or extending a plant metabolomics annotation resource:
  MS2 spectra and MS1 metabolite tables from public plant-relevant sources must be
  harmonized and merged into a single searchable database object.'
when_to_use_negative:
- The analysis targets non-plant metabolomes — the database is plant-specific and
  will lack non-plant metabolites
- The user already has a feature table and needs downstream statistics rather than
  database construction (use MetMiner's statistical/annotation panels instead)
- No programming-free interface is required and an existing general-purpose database
  (e.g. HMDB) suffices
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetMiner
  role: Pipeline whose metMiner-shinyapp hosts the compiled PM database and provides
    get_compound_info() for standardizing compound entries
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Provides the metID database-building utilities used to merge curated MS2 spectra
    and MS1 tables into 'databaseClass' objects
- name: GNPS
  role: Source of MS2 spectral data among the six public libraries merged into the
    PM database
- name: KEGG
  role: Plant-specific MS1 pathway source contributing 3,044 metabolites after filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:dfbb56d89ef5b8f9472acb4f5b8c8cedfdb4ac9ca950facc0ec3a6ee6bf5979b
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# plant-metabolite-source-curation

## Summary

Compile a plant-specific PM mass spectrometry database by converting six MS2 spectral libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and three filtered MS1 pathway tables (KEGG, KNApSAcK, PlantCyc) into metID 'databaseClass' objects for embedding in the metMiner-shinyapp.

## When to use

Use when building or extending a plant metabolomics annotation resource: MS2 spectra and MS1 metabolite tables from public plant-relevant sources must be harmonized and merged into a single searchable database object.

## When NOT to use

- The analysis targets non-plant metabolomes — the database is plant-specific and will lack non-plant metabolites
- The user already has a feature table and needs downstream statistics rather than database construction (use MetMiner's statistical/annotation panels instead)
- No programming-free interface is required and an existing general-purpose database (e.g. HMDB) suffices

## Inputs

- MS2 spectra files from six public libraries (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS)
- MS1 metabolite tables from KEGG (3,044 metabolites), KNApSAcK (1,627) and PlantCyc (2,800)
- Compound metadata fields: compound identifiers, exact masses, polarity, MS2 peak lists

## Outputs

- A single plant-specific PM database exported as a structured file loadable as a metID 'databaseClass' object
- Per-source metabolite count summary table (KEGG 3,044; KNApSAcK 1,627; PlantCyc 2,800)
- PM database embedded in the metMiner-shinyapp for metabolite annotation

## How to apply

Download the six MS2 spectral datasets (MoNA, MassBank, ReSpect, PlaSMA, NIST, GNPS) and the MS1 metabolite tables from the plant pathway sources KEGG (3,044 metabolites), KNApSAcK (1,627) and PlantCyc (2,800). Standardize each entry (compound identifiers, exact masses, polarity, MS2 peak lists) using the MetMiner accessor get_compound_info() and the metID database-building utilities from tidyMass. Build one metID 'databaseClass' object per source, then merge the curated MS2 spectra and MS1 tables into a single plant-specific PM database and export it as a structured file. Finally, tabulate the number of metabolites contributed by each source and embed the database in the metMiner-shinyapp to speed up and improve metabolite annotation.

## Related tools

- **MetMiner** (Pipeline whose metMiner-shinyapp hosts the compiled PM database and provides get_compound_info() for standardizing compound entries) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Provides the metID database-building utilities used to merge curated MS2 spectra and MS1 tables into 'databaseClass' objects)
- **GNPS** (Source of MS2 spectral data among the six public libraries merged into the PM database)
- **KEGG** (Plant-specific MS1 pathway source contributing 3,044 metabolites after filtering)

## Evaluation signals

- The exported database file exists and loads successfully as a metID 'databaseClass' object
- The per-source count table reproduces the expected totals: KEGG 3,044, KNApSAcK 1,627, PlantCyc 2,800 metabolites
- Entries carry standardized fields (compound identifiers, exact masses, polarity, MS2 peak lists) accessible via get_compound_info()
- The PM database is selectable/usable as an annotation database inside the metMiner-shinyapp
- Both MS1 and MS2 components are present in the merged database, mirroring the article's integrated design

## Limitations

- Suspected in-source fragmentation products may still be annotated as genuine metabolites; the authors state 'there is no satisfactory solution to detect and remove such redundant annotations' in the current version
- No release DOI, version number, or containerized deployment is documented for the pipeline, complicating exact reproducibility of the database build
- The database is plant-specific, so annotation accuracy gains do not generalize to non-plant samples

## Evidence

- [abstract] Plant-specific MS databases integrated to boost annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] MS1 and MS2 spectra both integrated: "Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [discussion] PM database purpose in MetMiner: [evidence span withheld — anchor status: not_found]
- [intro] tidyMass drives upstream processing: "The upstream data processing (peak picking and data cleaning) is driven by tidyMass, inheriting the advantages of reproducibility, traceability, and transparency."
