---
name: massive-dataset-molecular-networking-integration
description: Use when a paired omics analysis needs to pull GNPS molecular networking
  output for a MassIVE accession (MSV*) and connect it to genome mining results inside
  NPLinker.
when_to_use_negative:
- The dataset is not paired — there is no genome/BGC data to link the GNPS spectra
  to, so NPLinker integration adds nothing
- The task is standalone chemical class prediction of spectra (e.g. CANOPUS/NPClassifier
  only) without correlation-based BGC linking
- Genomes lack RefSeq or GenBank identifiers — per the article, genome data is only
  downloaded from NCBI when such identifiers are available
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: NPLinker
  role: Platform that downloads/extracts GNPS molecular networking data (GNPSDownloader,
    GNPSExtractor) and computes Metcalf scores over loaded GCFs and spectra
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Source of molecular networking output for the MassIVE accessions (MSV*) that
    NPLinker ingests
- name: BiG-SCAPE
  role: Provides gene cluster families that are loaded alongside the GNPS data in
    NPLinker
- name: antiSMASH
  role: Supplies BGC gbk files from the genomes corresponding to the GNPS datasets
    (antiSMASH 6 merged with prior antiSMASH 3 data)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:b4a90e801d13d86e7b7c0e58a0c79db20527b87e65d35e3f173b61f1e5995fcb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# massive-dataset-molecular-networking-integration

## Summary

Integrates MassIVE/GNPS molecular networking data (accessions such as MSV000084950, MSV000084771, MSV000078836) into the NPLinker platform so paired metabolomics and genomics datasets from the PoDP can be loaded, scored, and filtered for BGC–MS/MS spectrum linking.

## When to use

Use when a paired omics analysis needs to pull GNPS molecular networking output for a MassIVE accession (MSV*) and connect it to genome mining results inside NPLinker.

## When NOT to use

- The dataset is not paired — there is no genome/BGC data to link the GNPS spectra to, so NPLinker integration adds nothing
- The task is standalone chemical class prediction of spectra (e.g. CANOPUS/NPClassifier only) without correlation-based BGC linking
- Genomes lack RefSeq or GenBank identifiers — per the article, genome data is only downloaded from NCBI when such identifiers are available

## Inputs

- MassIVE/GNPS accession (MSV*) for a paired metabolomics dataset, e.g. MSV000084950 or MSV000084771
- GNPS molecular networking output (mgf and network files) downloaded via GNPSDownloader/GNPSExtractor
- nplinker.toml configuration file for NPLinker
- Genome mining results from antiSMASH (versions 3 and 6 merged) and BiG-SCAPE (GCFs at cut-off 0.3)
- mzXML files with mappings from filenames to spectrum ids

## Outputs

- Extracted GNPS molecular networking data loaded into an NPLinker object (spectra, GCFs, strain mappings)
- Link graph of candidate BGC–GCF to MS/MS spectrum links from npl.get_links (e.g. Metcalf scores)
- Mappings from mzXML filenames to spectrum ids for downstream scoring

## How to apply

Retrieve the paired datasets via their PoDP/MassIVE accessions (e.g. MSV000084950 for Cyanobacteria, MSV000084771 for Nocardia, and the MSV000078836/MSV000078839/MSV000079284 accessions for the Crüsemann Streptomyces/Salinispora data), downloading and extracting the GNPS data with NPLinker's GNPSDownloader and GNPSExtractor. Load the configuration with load_config on a nplinker.toml file and call npl.load_data() to bring the GNPS molecular networking output together with antiSMASH/BiG-SCAPE genome mining results (BiG-SCAPE run with a cut-off of 0.3). If needed, use extract_mappings_ms_filename_spectrum_id to map mzXML files to spectrum ids. Then compute standardised Metcalf scores on the loaded link graph via npl.get_links, applying the 2.5 cut-off (2.0 for the Nocardia dataset) before downstream NPClassScore filtering at 0.25.

## Related tools

- **NPLinker** (Platform that downloads/extracts GNPS molecular networking data (GNPSDownloader, GNPSExtractor) and computes Metcalf scores over loaded GCFs and spectra) — github.com/NPLinker/nplinker
- **GNPS** (Source of molecular networking output for the MassIVE accessions (MSV*) that NPLinker ingests)
- **BiG-SCAPE** (Provides gene cluster families that are loaded alongside the GNPS data in NPLinker)
- **antiSMASH** (Supplies BGC gbk files from the genomes corresponding to the GNPS datasets (antiSMASH 6 merged with prior antiSMASH 3 data))

## Examples

```
from nplinker import load_config, load_data; config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- All expected MSV accessions (e.g. MSV000084950, MSV000084771) download and extract without missing GNPS network or mgf files
- npl.load_data() populates both the GCF list (BiG-SCAPE cut-off 0.3) and the metabolomics spectra used in npl.get_links
- Standardised Metcalf scoring on the loaded data reproduces the reported 5/26 validated-link failures (75% full-workflow accuracy) with the 2.5 cut-off (2.0 for Nocardia)
- Mappings from mzXML filenames to spectrum ids cover the spectra required for the 24 detectable validated links (2 of 26 had missing spectra)

## Limitations

- Two validated compounds (retimycin, nocardimicin) could not be matched to MS/MS spectra in the datasets
- Genome data is only downloaded when RefSeq or GenBank identifiers are available, and strain mappings may need to be combined across multiple PoDP accessions
- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring the threshold to be lowered from 2.5 to 2

## Evidence

- [discussion] The Cyanobacteria and Nocardia datasets were accessed through their PoDP accessions, MSV000084950 and MSV000084771: "The Cyanobacteria and Nocardia datasets were accessed through their PoDP accessions, MSV000084950 and MSV000084771, respectively"
- [methods] It provides the tools to download and extract the GNPS data: "It provides the tools [`GNPSDownloader`][nplinker.metabolomics.gnps.GNPSDownloader] and [`GNPSExtractor`][nplinker.metabolomics.gnps.GNPSExtractor] to download and extract the GNPS data"
- [methods] you may need the function to extract the mappings from mzXML files to the spectrum ids: "you may need the function [extract_mappings_ms_filename_spectrum_id][nplinker.metabolomics.utils.extract_mappings_ms_filename_spectrum_id] to extract the mappings from mzXML files to the spectrum ids"
- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] first run through the docker version with a BiG-SCAPE cut-off of 0.3: "first run through the docker version with a BiG-SCAPE cut-off of 0.3"
