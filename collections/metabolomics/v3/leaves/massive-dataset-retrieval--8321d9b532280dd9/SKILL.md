---
name: massive-dataset-retrieval
description: Use when the analysis goal requires re-analyzing archived public untargeted
  metabolomics raw files (mzML/mzXML) from a named MassIVE accession or public dataset
  repository, especially where no clinical records exist (environmental wastewater,
  food samples, retrospective cohorts).
when_to_use_negative:
- The dataset of interest is already locally available and pre-annotated (retrieval
  is redundant)
- The goal is immediate clinical decision-making — the GNPS Drug Library readouts
  are explicitly not intended for that purpose
- Only targeted metabolomics data or a curated feature table is available and no MS/MS
  spectra need re-matching
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Hosts the Drug Library reference MS/MS spectra and the library search (cosine
    score, mass-bmatch tolerance) used to annotate retrieved sample spectra; MassIVE
    stores the raw public datasets
- name: microbe_masst / domainMASSTs
  role: jobs.py batch-searches multiple MS/MS spectra via the GNPS Fast Search API
    against indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and
    NORMAN, producing _matches.tsv, _library.tsv, and _datasets.tsv outputs
  repo: https://github.com/robinschmid/microbe_masst
- name: Manuscript_GNPS_Drug_Library
  role: Manuscript repository containing code and data for the GNPS Drug Library drug-exposure
    readout workflow applied to the retrieved datasets
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:a6776dd8653cbcd09bebe03463f03395b8678928422c64f79f539c0a03239ed3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - Manuscript_GNPS_Drug_Library
  - MassIVE
  - microbe_masst / domainMASSTs
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# massive-dataset-retrieval

## Summary

Retrieves archived public untargeted metabolomics data (mzML/mzXML files) from MassIVE accessions and domain datasets (e.g., wastewater influent MSV000097575, Global FoodOmics) so they can be re-analyzed against reference MS/MS libraries such as the GNPS Drug Library to establish exposures retroactively.

## When to use

Use when the analysis goal requires re-analyzing archived public untargeted metabolomics raw files (mzML/mzXML) from a named MassIVE accession or public dataset repository, especially where no clinical records exist (environmental wastewater, food samples, retrospective cohorts).

## When NOT to use

- The dataset of interest is already locally available and pre-annotated (retrieval is redundant)
- The goal is immediate clinical decision-making — the GNPS Drug Library readouts are explicitly not intended for that purpose
- Only targeted metabolomics data or a curated feature table is available and no MS/MS spectra need re-matching

## Inputs

- MassIVE accession identifier (e.g., MSV000097575) with raw mzML/mzXML files and sample metadata
- Global FoodOmics untargeted metabolomics files (~3500 foods/beverages)
- GNPS Drug Library reference MS/MS spectra with pharmacologic metadata
- USI list or MZmine-generated .mgf file for batch Fast Search API queries

## Outputs

- Per-sample MS/MS annotation table mapping detected compounds to reference spectra (drug_annotations.csv)
- Drug-by-sample matrix with pharmacologic class and exposure source per annotation
- Aggregated detection counts (e.g., per-compound detections by collection month or per food category)
- MASST-style output files (_matches.tsv, _library.tsv, _datasets.tsv, _count_domain.tsv) when using jobs.py

## How to apply

Identify the MassIVE accession (e.g., MSV000097575, n=55 wastewater influent samples collected March–June 2020) or repository dataset (e.g., Global FoodOmics, ~3500 foods/beverages) and download the mzML/mzXML files plus sample metadata. Load the appropriate reference MS/MS library — for drug exposure work, the GNPS Drug Library with drugs, metabolites/analogs and machine-readable pharmacologic metadata — and match sample spectra via spectral similarity search using cosine score with mass-bmatch tolerance per GNPS library search conventions. Alternatively, use the microbe_masst jobs.py batch script, which leverages the GNPS Fast Search API to search spectra against indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN; note that the Fast Search API can fail entries, so re-run with skip_existing=True until no new output is generated. Aggregate per-sample annotations into a drug-by-sample table (drug_annotations.csv) with pharmacologic class and exposure source, then compute domain-specific summaries (e.g., per-compound detection counts by collection month for wastewater seasonality, or detections per food category for FoodOmics).

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Hosts the Drug Library reference MS/MS spectra and the library search (cosine score, mass-bmatch tolerance) used to annotate retrieved sample spectra; MassIVE stores the raw public datasets)
- **microbe_masst / domainMASSTs** (jobs.py batch-searches multiple MS/MS spectra via the GNPS Fast Search API against indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, producing _matches.tsv, _library.tsv, and _datasets.tsv outputs) — https://github.com/robinschmid/microbe_masst
- **Manuscript_GNPS_Drug_Library** (Manuscript repository containing code and data for the GNPS Drug Library drug-exposure readout workflow applied to the retrieved datasets) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Evaluation signals

- MSV000097575 download resolves to n=55 wastewater influent samples collected March–June 2020 in mzML/mzXML format
- Retrospective drug readout reproduces dextromethorphan detection in wastewater influent with declining detections (dextromethorphan and antibiotics) from March to June 2020
- FoodOmics re-analysis tabulates ampicillin and tetracycline detections in fish/beef/turkey and spinosad/thiabendazole detections in produce categories
- Final drug_annotations.csv contains per-sample annotations joined to pharmacologic class and exposure source metadata
- When using jobs.py, re-runs with skip_existing=True eventually produce no new output, indicating all Fast Search API entries completed

## Limitations

- The GNPS Drug Library is not intended for immediate clinical decision-making; readouts are exploratory exposure evidence, not clinical records
- The GNPS Fast Search API intermittently fails entries, requiring multiple re-runs of jobs.py (with skip_existing=True) until all matches are collected
- Existing MS/MS reference libraries historically capture only parent drugs, so metabolized drug versions present in samples can be missed unless the library includes drug metabolites/analogs

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] This resource enables data science analysis to empirically - and retroactively - determine drug exposures using untargeted metabolomics, supporting research across disciplines.: "This resource enables data science analysis to empirically - and retroactively - determine drug exposures using untargeted metabolomics, supporting research across disciplines."
- [abstract] While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable.: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
- [readme] execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN: [evidence span withheld — anchor status: external_doc]
- [intro] They frequently overlook over-the-counter medications and dietary supplements, and often fail to account for patient adherence.: "They frequently overlook over-the-counter medications and dietary supplements, and often fail to account for patient adherence."
