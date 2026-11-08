---
name: gnps-molecular-networking-data-handling
description: Use when a paired-omics linking analysis requires GNPS molecular networking
  output to be downloaded, extracted, and mapped to spectrum identifiers for subsequent
  scoring.
when_to_use_negative:
- When the analysis already has an annotated spectra.mgf and only needs ontology-level
  class comparison (e.g. CANOPUS vs MolNetEnhancer discrepancy analysis on already-annotated
  spectra)
- When the task is purely genome-side, e.g. BiG-SCAPE GCF clustering or antiSMASH
  BGC detection, without any metabolomics component
- When the user only wants to visualise a CANOPUS treemap from an already-computed
  Sirius projectspace (canopus_treemap covers that need)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Platform that orchestrates the whole linking workflow; provides load_config,
    load_data, get_links and the GNPSDownloader/GNPSExtractor helpers used to handle
    GNPS molecular networking data
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Source of the molecular networking output (MGF, metadata, networks) that is
    downloaded and extracted; the spectra.mgf it produces is the direct input for
    CANOPUS-based class prediction and for mapping spectra to mzXML filenames
- name: CANOPUS
  role: Downstream consumer of the extracted spectra.mgf; run via Sirius v4.9.3 on
    the GNPS molecular networking MGF file with 'formula zodiac structure canopus'
    and --maxmz 850 to predict ClassyFire/NPClassifier classes for spectra below 850
    Da
- name: MolNetEnhancer
  role: External ontology-prediction tool that must be run (e.g. on the GNPS platform)
    to complement CANOPUS; its output is ingested for comparison and for MF-level
    class annotation
- name: ClassyFire
  role: One of the chemical ontology frameworks used to classify the spectra and MIBiG
    products; its classifications are retrieved through the GNPS API for MIBiG entries
    and predicted by CANOPUS/MolNetEnhancer for spectra
- name: NPClassifier
  role: Alternative chemical ontology used for spectra and MIBiG classifications;
    per-spectrum NPClassifier predictions use a probability cut-off of 0.33
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:746e8c2516d93e56f8c8fbd1b761230568488678238a28b36759b836b74c0b64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ClassyFire
  - GNPS
  - MolNetEnhancer
  - NPClassifier
  - NPLinker
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# GNPS molecular networking data handling

## Summary

Skill for handling and integrating Global Natural Products Social Molecular Networking (GNPS) molecular networking data within the NPLinker platform, including downloading, extracting, and mapping MS/MS spectra files for downstream linking.

## When to use

Use when a paired-omics linking analysis requires GNPS molecular networking output to be downloaded, extracted, and mapped to spectrum identifiers for subsequent scoring.

## When NOT to use

- When the analysis already has an annotated spectra.mgf and only needs ontology-level class comparison (e.g. CANOPUS vs MolNetEnhancer discrepancy analysis on already-annotated spectra)
- When the task is purely genome-side, e.g. BiG-SCAPE GCF clustering or antiSMASH BGC detection, without any metabolomics component
- When the user only wants to visualise a CANOPUS treemap from an already-computed Sirius projectspace (canopus_treemap covers that need)

## Inputs

- GNPS molecular networking job URL or MassIVE accession (e.g. MSV000078836, MSV000078839, MSV000079284, MSV000084950, MSV000084771)
- nplinker.toml configuration file
- mzXML files with MS/MS spectra
- spectra.mgf file produced by GNPS molecular networking
- molecular_families.tsv (GCF-family mapping)
- paired omics metadata from the Paired Omics Data Platform (PoDP)

## Outputs

- Extracted GNPS dataset directory (spectra.mgf, metadata, network edges)
- Mappings between mzXML filenames and spectrum ids
- spectra.mgf suitable for CANOPUS/Sirius input
- class_annotations.tsv (per-spectrum and per-MF ClassyFire/NPClassifier annotations)
- Ranked BGC-MS/MS link candidates after Metcalf and NPClassScore filtering

## How to apply

The skill grounds on the NPLinker Python API: after instantiating the NPLinker object, the user loads configuration via load_config('path/to/nplinker.toml'), then calls npl.load_data() to ingest all paired omics data. For GNPS data specifically, the platform provides the GNPSDownloader and GNPSExtractor tools to download and extract the GNPS molecular networking data. The extracted data includes the spectra.mgf file (produced by GNPS molecular networking), which is then used as the basis for downstream processing like CANOPUS-based structure-class prediction (Sirius v4.9.3 with 'formula zodiac structure canopus' and --maxmz 850) and for molecular networking spectra metadata. When mapping the MS/MS spectra back to raw data, the helper extract_mappings_ms_filename_spectrum_id extracts mappings from mzXML files to the spectrum ids, enabling per-spectrum class annotations (class_annotations.tsv) at both per-spectrum and per-molecular-family level. The rationale is that molecular networking via GNPS provides the standard container (MGF files, edge/component metadata) that downstream class-prediction and linking tools (CANOPUS, MolNetEnhancer, NPClassScore) require; without the correct download/extract/mapping chain the spectra cannot be keyed to genome-derived BGC-GCF candidates.

## Related tools

- **NPLinker** (Platform that orchestrates the whole linking workflow; provides load_config, load_data, get_links and the GNPSDownloader/GNPSExtractor helpers used to handle GNPS molecular networking data) — github.com/NPLinker/nplinker
- **GNPS** (Source of the molecular networking output (MGF, metadata, networks) that is downloaded and extracted; the spectra.mgf it produces is the direct input for CANOPUS-based class prediction and for mapping spectra to mzXML filenames)
- **CANOPUS** (Downstream consumer of the extracted spectra.mgf; run via Sirius v4.9.3 on the GNPS molecular networking MGF file with 'formula zodiac structure canopus' and --maxmz 850 to predict ClassyFire/NPClassifier classes for spectra below 850 Da)
- **MolNetEnhancer** (External ontology-prediction tool that must be run (e.g. on the GNPS platform) to complement CANOPUS; its output is ingested for comparison and for MF-level class annotation)
- **ClassyFire** (One of the chemical ontology frameworks used to classify the spectra and MIBiG products; its classifications are retrieved through the GNPS API for MIBiG entries and predicted by CANOPUS/MolNetEnhancer for spectra)
- **NPClassifier** (Alternative chemical ontology used for spectra and MIBiG classifications; per-spectrum NPClassifier predictions use a probability cut-off of 0.33)

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- class_annotations.tsv exists and is a well-formed TSV with expected class-level columns and per-spectrum and per-MF rows
- MF-level class entries only contain classes meeting the 20% occurrence threshold within the molecular family
- Extracted GNPS directory contains the expected spectra.mgf, metadata and network files
- The mappings from mzXML filenames to spectrum ids cover all spectra referenced in the MGF
- The number of candidate links per GCF after NPClassScore filtering at cut-off 0.25 is reduced by on average 63% relative to co-occurrence-only Metcalf scoring
- For the Streptomyces/Salinispora dataset, the superclass-count table reproduces the reported 2686-vs-1301 lipid superclass discrepancy for MolNetEnhancer vs CANOPUS over 6606 spectra

## Limitations

- CANOPUS depends on calculating fragmentation trees, which is time-consuming, so it is only used for lower-mass compounds below 850 Da
- MolNetEnhancer has to be run externally (e.g. on the GNPS platform) and cannot currently be run within NPLinker
- Discarding candidate links without structure-based classes (filter_missing_scores toggle) affects results: not filtering such links results in an average decrease of 49% of the candidate links per GCF
- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction, so those links cannot receive an NPClassScore and are discarded by default
- The choice of ontology-prediction tool strongly influences outcomes: CANOPUS and MolNetEnhancer disagree substantially on ClassyFire superclass predictions (e.g. 'Lipids and lipid-like molecules' predicted 2686 times by MolNetEnhancer vs 1301 by CANOPUS)

## Evidence

- [methods] GNPS molecular networking data download and extraction: "It provides the tools [`GNPSDownloader`][nplinker.metabolomics.gnps.GNPSDownloader] and [`GNPSExtractor`][nplinker.metabolomics.gnps.GNPSExtractor] to download and extract the GNPS data"
- [discussion] SIRIUS/CANOPUS run on GNPS MGF: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [discussion] CANOPUS mass threshold: [evidence span withheld — anchor status: not_found]
- [readme] CANOPUS command-line settings: [evidence span withheld — anchor status: not_found]
- [readme] Per-spectrum class annotations output: [evidence span withheld — anchor status: not_found]
- [discussion] MolNetEnhancer external run: "Currently, MolNetEnhancer has to be run externally, for example on the GNPS platform"
- [discussion] Filtering missing structure-based classes: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [abstract] NPClassScore performance: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [readme] CANOPUS vs MolNetEnhancer discrepancy: [evidence span withheld — anchor status: not_found]
- [discussion] ClassyFire retrieval via GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
