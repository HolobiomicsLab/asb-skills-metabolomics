---
name: paired-omics-dataset-configuration
description: Use when a paired genome–metabolome dataset with a Paired Omics Data
  Platform (PoDP) accession (e.g. MSV000078836, MSV000084950, MSV000084771) must be
  assembled in NPLinker for BGC–MS/MS spectrum linking.
when_to_use_negative:
- The dataset has no Paired Omics Data Platform (PoDP) accession or paired genome–metabolome
  strain mapping
- You only need raw MS/MS processing (e.g. feature detection/mzML conversion) without
  linking to BGCs
- The genomes lack RefSeq or GenBank identifiers and cannot be retrieved from NCBI
  for antiSMASH/BiG-SCAPE
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3172
tools:
- name: NPLinker
  role: Platform that loads the podp-mode dataset via nplinker.toml, downloads and
    extracts GNPS data (GNPSDownloader/GNPSExtractor), and assembles GCFs and strain
    mappings
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Source of molecular networking output downloaded and extracted per dataset
    accession
- name: antiSMASH
  role: Provides BGC gbk files for each dataset's strains; antiSMASH 6 output merged
    with prior antiSMASH 3 data
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (GCFs) using a docker run with a cut-off
    of 0.3
- name: CANOPUS
  role: Run via Sirius v4.9.3 directly on the mgf file resulting from GNPS molecular
    networking within NPLinker
  repo: github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:be5dc54bc79b41d6b27b1eb8e4d28148ccd587145bdaabef2fbbf2313d2423cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - CANOPUS
  - GNPS
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# paired-omics-dataset-configuration

## Summary

Configure and load the three PoDP paired omics datasets (154 Streptomyces/Salinispora, 24 cyanobacterial, and 11 Nocardia strains) in NPLinker's podp mode via a nplinker.toml file, assembling GNPS molecular networking, antiSMASH/BiG-SCAPE GCFs, and strain mappings for BGC–MS/MS linking.

## When to use

Use when a paired genome–metabolome dataset with a Paired Omics Data Platform (PoDP) accession (e.g. MSV000078836, MSV000084950, MSV000084771) must be assembled in NPLinker for BGC–MS/MS spectrum linking.

## When NOT to use

- The dataset has no Paired Omics Data Platform (PoDP) accession or paired genome–metabolome strain mapping
- You only need raw MS/MS processing (e.g. feature detection/mzML conversion) without linking to BGCs
- The genomes lack RefSeq or GenBank identifiers and cannot be retrieved from NCBI for antiSMASH/BiG-SCAPE

## Inputs

- nplinker.toml configuration file (podp mode with the dataset's PoDP id)
- PoDP/MassIVE accession identifiers (MSV000078836, MSV000078839, MSV000079284, MSV000084950, MSV000084771)
- GNPS molecular networking output (mgf files)
- antiSMASH gbk files (antiSMASH 6 merged with antiSMASH 3 for 50 strains)
- BiG-SCAPE GCF clustering output (cut-off 0.3)
- mzXML files with strain/spectrum id mappings

## Outputs

- Loaded NPLinker object (npl) with npl.gcfs, spectra, and strain mappings
- Dataset-specific strain-to-genome and strain-to-spectrum mappings
- Candidate link graph ready for Metcalf/NPClassScore scoring

## How to apply

Instantiate the NPLinker class with a nplinker.toml configuration in podp mode using the dataset's PoDP id, then call load_config('path/to/nplinker.toml') and npl.load_data() to assemble GNPS molecular networking output, antiSMASH/BiG-SCAPE GCFs, and strain mappings. For the Streptomyces/Salinispora dataset, combine strain mappings across the three MassIVE accessions MSV000078836, MSV000078839, and MSV000079284; the Cyanobacteria and Nocardia datasets map to MSV000084950 and MSV000084771 respectively. Genomes are downloaded from NCBI only when RefSeq or GenBank identifiers are available, and antiSMASH 6 output is merged with prior antiSMASH 3 data for the other 50 strains; BiG-SCAPE is run via docker with a cut-off of 0.3. Use extract_mappings_ms_filename_spectrum_id to map mzXML files to spectrum ids so that MS/MS spectra can be tied to the correct strains.

## Related tools

- **NPLinker** (Platform that loads the podp-mode dataset via nplinker.toml, downloads and extracts GNPS data (GNPSDownloader/GNPSExtractor), and assembles GCFs and strain mappings) — github.com/NPLinker/nplinker
- **GNPS** (Source of molecular networking output downloaded and extracted per dataset accession)
- **antiSMASH** (Provides BGC gbk files for each dataset's strains; antiSMASH 6 output merged with prior antiSMASH 3 data)
- **BiG-SCAPE** (Groups BGCs into gene cluster families (GCFs) using a docker run with a cut-off of 0.3)
- **CANOPUS** (Run via Sirius v4.9.3 directly on the mgf file resulting from GNPS molecular networking within NPLinker) — github.com/kaibioinfo/canopus_treemap

## Examples

```
config = load_config('path/to/nplinker.toml'); npl = NPLinker(config); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- load_data() completes without errors and npl.gcfs, spectra, and strain mappings are populated for the dataset
- For the Streptomyces/Salinispora dataset, strain mappings correctly span all three accessions MSV000078836, MSV000078839, and MSV000079284
- BiG-SCAPE was run with the expected 0.3 cut-off and GCF counts are consistent with the merged antiSMASH 3 + antiSMASH 6 outputs
- mzXML-to-spectrum-id mappings extracted via extract_mappings_ms_filename_spectrum_id cover the spectra in the GNPS mgf
- Downstream Metcalf linking (cut-off 2.5, or 2 for Nocardia) reproduces candidate links per GCF consistent with the reported counts (27 Cyanobacteria, 206 Nocardia before NPClassScore filtering)

## Limitations

- Genome data is only downloaded from NCBI when strains have RefSeq or GenBank identifiers, so some strains may lack genome input
- Strain mappings for the Streptomyces/Salinispora dataset must be combined from three PoDP accessions, which is required for reproducibility
- Merged antiSMASH 3 and antiSMASH 6 outputs introduce version inconsistency across 50 strains
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally, for example on the GNPS platform

## Evidence

- [methods] It provides the tools GNPSDownloader and GNPSExtractor to download and extract the GNPS data: "It provides the tools [`GNPSDownloader`][nplinker.metabolomics.gnps.GNPSDownloader] and [`GNPSExtractor`]"
- [methods] config load and data load snippets: "config = load_config('path/to/nplinker.toml')"
- [discussion] the corresponding genomes were downloaded from NCBI if they had RefSeq or GenBank identifiers: "the corresponding genomes were downloaded from NCBI if they had RefSeq or GenBank identifiers"
- [discussion] first run through the docker version with a BiG-SCAPE cut-off of 0.3: "first run through the docker version with a BiG-SCAPE cut-off of 0.3"
- [discussion] Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [discussion] The Cyanobacteria and Nocardia datasets were accessed through their PoDP accessions, MSV000084950 and MSV000084771: "The Cyanobacteria and Nocardia datasets were accessed through their PoDP accessions, MSV000084950 and MSV000084771"
