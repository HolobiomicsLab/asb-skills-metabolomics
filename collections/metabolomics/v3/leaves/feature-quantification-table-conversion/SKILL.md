---
name: feature-quantification-table-conversion
description: Use when an OpenMS consensusXML/feature map from UmetaFlow pre-processing
  must be transformed into a GNPS-upload-ready feature quantification table alongside
  an MGF and sample metadata.
when_to_use_negative:
- The input is already a GNPS-ready feature quantification table rather than an OpenMS
  consensus map.
- Feature maps from tools other than UmetaFlow/OpenMS whose consensusXML schema is
  not compatible with pyOpenMS conversion.
- No MS2 spectra are available for the features, since GNPS FBMN/IIMN requires tandem
  spectra.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyOpenMS
  role: Python bindings to OpenMS algorithms used to convert the filtered consensus
    map into the GNPS Feature Quantification table
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: OpenMS
  role: FileFilter keeps only features with MS2 information before conversion; OpenMS
    algorithms handle consensus map reading/writing
- name: UmetaFlow
  role: Workflow that orchestrates the GNPSExport step (MGF, quantification table,
    supplementary table) after pre-processing
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:5dd72133459f4e907de48b3b3f5f2e3c381a6f7a3f8800a43e5d0d46d60acc32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/pyOpenMS_UmetaFlow
  tools:
  - OpenMS
  - Python
  - UmetaFlow
  - pyOpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# Feature quantification table conversion

## Summary

Converts an OpenMS consensus feature map into a GNPS Feature Quantification table (TXT) as part of the GNPSExport step in UmetaFlow, so feature intensities can feed Feature-Based or Ion Identity Molecular Networking jobs.

## When to use

Use when an OpenMS consensusXML/feature map from UmetaFlow pre-processing must be transformed into a GNPS-upload-ready feature quantification table alongside an MGF and sample metadata.

## When NOT to use

- The input is already a GNPS-ready feature quantification table rather than an OpenMS consensus map.
- Feature maps from tools other than UmetaFlow/OpenMS whose consensusXML schema is not compatible with pyOpenMS conversion.
- No MS2 spectra are available for the features, since GNPS FBMN/IIMN requires tandem spectra.

## Inputs

- OpenMS consensus feature map (consensusXML) from UmetaFlow pre-processing/re-quantification
- Filtered consensus map retaining only MS2-containing features
- dataset.tsv sample metadata table (sample_name and comment columns)
- mzML files for MS2 spectrum clustering

## Outputs

- GNPS Feature Quantification table (TXT)
- Clustered MS2 spectra in a single MGF file
- Adduct-connection supplementary table for IIMN collapsed ion-identity edges
- Sample metadata table for GNPS upload

## How to apply

After UmetaFlow's pre-processing and optional re-quantification steps produce a consensus feature map, first apply an OpenMS FileFilter to retain only features that carry MS2 information, since GNPS FBMN/IIMN needs tandem spectra per feature. Then use pyOpenMS (via the UmetaFlow GNPSExport notebook) to convert the filtered consensus map into the Feature Quantification table (TXT) and to cluster the MS2 spectra into a single MGF file. Generate the comma-separated supplementary table carrying adduct-connection information for IIMN collapsed ion-identity edges, and build the sample metadata table from the dataset.tsv entries (sample_name/comment columns). Validate the output set by checking that the MGF, quantification table, adduct-connection table and metadata table are all present in upload-ready format before creating a FBMN or IIMN job at GNPS.

## Related tools

- **pyOpenMS** (Python bindings to OpenMS algorithms used to convert the filtered consensus map into the GNPS Feature Quantification table) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **OpenMS** (FileFilter keeps only features with MS2 information before conversion; OpenMS algorithms handle consensus map reading/writing)
- **UmetaFlow** (Workflow that orchestrates the GNPSExport step (MGF, quantification table, supplementary table) after pre-processing) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
python 4_GNPSExport.ipynb  # or snakemake --use-conda --cores all after enabling the GNPS rule in config.yaml
```

## Evaluation signals

- All four GNPS-required files (MGF, quantification TXT, adduct-connection table, metadata table) are present in upload-ready format.
- Quantification table row counts match the number of consensus features retained after MS2 filtering.
- Each row of the quantification table maps to a feature spectrum present in the MGF.
- Sample metadata table columns mirror the dataset.tsv sample_name/comment structure.
- Resulting FBMN/IIMN GraphML network, once generated, can be annotated with SIRIUS/CSI:FingerID predictions without missing feature ids.

## Limitations

- Missing MS2 information for a feature means it is excluded by the FileFilter step and will not appear in the quantification table or MGF.
- The output is specific to GNPS FBMN/IIMN upload formats and cannot be directly reused by other networking tools without adaptation.
- OpenMS version compatibility matters — the workflow pins OpenMS 3.0 (or 3.3.0 on macOS) and pyOpenMS bindings must match.
- The adduct-connection supplementary table is only meaningful when Ion Identity Molecular Networking with collapsed ion-identity edges is intended.

## Evidence

- [methods] GNPSExport generates quantification table for FBMN: [evidence span withheld — anchor status: not_found]
- [methods] FileFilter retains MS2-containing features: [evidence span withheld — anchor status: not_found]
- [methods] Adduct-connection table for IIMN edges: [evidence span withheld — anchor status: not_found]
- [readme] Metadata derived from dataset.tsv: "Add all your files in the `data/raw/` or `data/mzML/` directory and generate the `dataset.tsv` table to specify the samples (filenames) that will be processed."
- [readme] Notebook 4 produces all GNPS files: [evidence span withheld — anchor status: external_doc]
- [intro] OpenMS 3.0 toolchain referenced: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
