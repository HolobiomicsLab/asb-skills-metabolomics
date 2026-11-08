---
name: ms2-spectrum-clustering-mgf-generation
description: Use when a processed LC-MS/MS feature table (OpenMS consensusXML) must
  be exported for GNPS Feature-Based Molecular Networking or Ion Identity Molecular
  Networking, i.e. you need an MGF plus quantification and metadata files for a GNPS
  upload.
when_to_use_negative:
- When the consensus feature map has not yet been aligned and linked across samples
  — clustering requires aligned features with MS2 information.
- When the goal is pure spectral library matching on an existing MGF rather than building
  a new GNPS FBMN/IIMN export set.
- When the downstream tool is not GNPS FBMN/IIMN (e.g. MZmine2 or XCMS export formats
  are expected instead).
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPSExport
  role: OpenMS tool that clusters the MS2 information of the filtered consensus map
    into a single MGF file and converts the consensus file into a Feature Quantification
    table (TXT) and comma-separated supplementary table.
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS FileFilter
  role: Applied to the consensus feature map to retain only features that carry MS2
    information before GNPSExport.
- name: pyOpenMS
  role: Python bindings used to convert the filtered consensus map into a GNPS Feature
    Quantification table (TXT) and to generate the adduct-connection and sample metadata
    tables.
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: GNPS
  role: Downstream service that consumes the MGF, quantification table, adduct-connection
    table, and metadata table to run FBMN or IIMN jobs and return a GraphML network.
- name: UmetaFlow
  role: Snakemake/pyOpenMS workflow that orchestrates the GNPS export step (4_GNPSExport)
    as part of the LC-MS/MS pipeline.
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:b533de677853e0f3702f69f80e14941d0d5cbcda1996569b6e160bb7e716b9bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - GNPS
  - GNPSExport
  - OpenMS
  - OpenMS FileFilter
  - Python
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# MS2 spectrum clustering and MGF generation for GNPS molecular networking

## Summary

Clusters MS2 spectra from a filtered OpenMS consensus feature map into a single MGF file and generates the accompanying feature quantification, adduct-connection, and metadata tables needed to run GNPS FBMN/IIMN molecular networking jobs.

## When to use

Use when a processed LC-MS/MS feature table (OpenMS consensusXML) must be exported for GNPS Feature-Based Molecular Networking or Ion Identity Molecular Networking, i.e. you need an MGF plus quantification and metadata files for a GNPS upload.

## When NOT to use

- When the consensus feature map has not yet been aligned and linked across samples — clustering requires aligned features with MS2 information.
- When the goal is pure spectral library matching on an existing MGF rather than building a new GNPS FBMN/IIMN export set.
- When the downstream tool is not GNPS FBMN/IIMN (e.g. MZmine2 or XCMS export formats are expected instead).

## Inputs

- OpenMS consensus feature map (consensusXML) from aligned/linked features, filtered to features with MS2 information
- All mzML files corresponding to the analyzed samples
- dataset.tsv table specifying sample names (e.g. ISP2_blank, NBC_00162)

## Outputs

- Single MGF file containing clustered MS2 spectra
- GNPS Feature Quantification table (TXT)
- Adduct-connection supplementary table (comma-separated) for IIMN collapsed ion-identity edges
- Sample metadata table for GNPS upload
- Annotated GNPS GraphML network with integrated SIRIUS/CSI:FingerID predictions

## How to apply

First apply an OpenMS FileFilter to the consensus feature map to retain only features that carry MS2 information, since GNPS clusters spectra rather than raw features. Run OpenMS GNPSExport on the filtered map together with all mzML files: it clusters the MS2 information into a single MGF file, converts the consensus map into a Feature Quantification table (TXT), and produces a comma-separated supplementary table. Generate the adduct-connection supplementary table to support IIMN networks with collapsed ion-identity edges, and build the sample metadata table from the dataset.tsv sample definitions. After the GNPS job returns a network, parse the downloaded GraphML file and integrate the #1 ranked SIRIUS/CSI:FingerID predictions into it for visualization. Note that this export sits after pre-processing (feature detection, alignment, grouping) and optional re-quantification in the UmetaFlow pipeline, so the consensus map must already be aligned across samples.

## Related tools

- **GNPSExport** (OpenMS tool that clusters the MS2 information of the filtered consensus map into a single MGF file and converts the consensus file into a Feature Quantification table (TXT) and comma-separated supplementary table.) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS FileFilter** (Applied to the consensus feature map to retain only features that carry MS2 information before GNPSExport.)
- **pyOpenMS** (Python bindings used to convert the filtered consensus map into a GNPS Feature Quantification table (TXT) and to generate the adduct-connection and sample metadata tables.) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **GNPS** (Downstream service that consumes the MGF, quantification table, adduct-connection table, and metadata table to run FBMN or IIMN jobs and return a GraphML network.)
- **UmetaFlow** (Snakemake/pyOpenMS workflow that orchestrates the GNPS export step (4_GNPSExport) as part of the LC-MS/MS pipeline.) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
snakemake --use-conda --cores all  # rule GNPSExport in snakemake_UmetaFlow
```

## Evaluation signals

- All GNPS-required files (MGF, feature quantification table, adduct-connection table, metadata table) are present in upload-ready format before submitting the job.
- The MGF contains MS2 spectra only for features that survived the FileFilter step, i.e. features with MS2 information.
- The feature quantification table (TXT) rows correspond 1:1 with consensus features and columns with the sample names listed in dataset.tsv.
- The adduct-connection table is comma-separated and links adducts belonging to the same compound for IIMN collapsed ion-identity edges.
- After the GNPS job, the downloaded GraphML contains SIRIUS/CSI:FingerID #1 ranked predictions integrated into node attributes (MSI level 2/3).

## Limitations

- The export depends on upstream pre-processing quality; missing values that were not re-quantified will be absent from the quantification table.
- Vendor raw file conversion must already be done (Thermo raw files are converted to centroided mzML; Agilent/Bruker files must be converted independently via proteowizard).
- SIRIUS integration into the GraphML is limited to MSI level 2/3 annotations — it does not guarantee structural certainty.

## Evidence

- [methods] GNPSExport clusters MS2 information into a single MGF file: [evidence span withheld — anchor status: not_found]
- [methods] FileFilter retains features with MS2 information: [evidence span withheld — anchor status: not_found]
- [methods] Adduct-connection supplementary table for IIMN: [evidence span withheld — anchor status: not_found]
- [readme] Step 4 of pyOpenMS_UmetaFlow generates FBMN/IIMN job files: [evidence span withheld — anchor status: external_doc]
- [readme] Graph view integrates SIRIUS predictions into network GraphML: "**Graph view**: Integrate SIRIUS predictions to the network (GraphML) and GNPS library annotations to the feature matrix - MSI level 2 (optional)."
- [intro] UmetaFlow uses OpenMS 3.0 for feature linking and spectral matching: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
