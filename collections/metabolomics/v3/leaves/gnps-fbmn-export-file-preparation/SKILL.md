---
name: gnps-fbmn-export-file-preparation
description: Use when a processed LC-MS/MS feature map (consensusXML after feature
  detection, alignment, adduct annotation, and feature linking) must be exported to
  run a GNPS FBMN or IIMN job downstream.
when_to_use_negative:
- When the raw vendor data has not yet been converted to mzML or the pre-processing
  workflow (feature detection, alignment, adduct annotation, feature linking) has
  not been run — no consensus feature map exists to export.
- When downstream analysis uses SIRIUS/MS2Query annotations directly without a GNPS
  FBMN/IIMN network job — the export file set is not needed.
- When spectra already exist as a GNPS-ready MGF from another tool; regenerating via
  GNPSExport would duplicate the clustering step.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: OpenMS GNPSExport
  role: Clusters MS2 information from the filtered consensus feature map into a single
    MGF file for GNPS
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: OpenMS FileFilter
  role: Filters the consensus feature map to retain only features that carry MS2 information
    before export
- name: pyOpenMS
  role: Converts the filtered consensus map into the GNPS Feature Quantification table
    (TXT) and generates the adduct-connection supplementary table
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: GNPS
  role: Consumes the generated file set to run FBMN or IIMN jobs and produce the GraphML
    network
- name: SIRIUS
  role: 'Provides #1-ranked molecular formula and structural predictions that are
    integrated into the GNPS GraphML network'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:1f8019866ceda3e862d530556bc853b8c250c596a8e3a6959dbd0305e1cc6001
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - GNPS
  - OpenMS
  - OpenMS FileFilter
  - OpenMS GNPSExport
  - Python
  - SIRIUS
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# GNPS FBMN export file preparation

## Summary

Prepares the exact file set GNPS requires for Feature-Based Molecular Networking and Ion Identity Molecular Networking jobs by filtering the OpenMS consensus feature map to features carrying MS2 spectra, then generating the MGF, feature quantification table, adduct-connection supplementary table, and metadata table via OpenMS GNPSExport and pyOpenMS.

## When to use

Use when a processed LC-MS/MS feature map (consensusXML after feature detection, alignment, adduct annotation, and feature linking) must be exported to run a GNPS FBMN or IIMN job downstream.

## When NOT to use

- When the raw vendor data has not yet been converted to mzML or the pre-processing workflow (feature detection, alignment, adduct annotation, feature linking) has not been run — no consensus feature map exists to export.
- When downstream analysis uses SIRIUS/MS2Query annotations directly without a GNPS FBMN/IIMN network job — the export file set is not needed.
- When spectra already exist as a GNPS-ready MGF from another tool; regenerating via GNPSExport would duplicate the clustering step.

## Inputs

- OpenMS consensus feature map (consensusXML) with adduct annotations and linked MS2 spectra
- All mzML files for the dataset
- dataset.tsv sample metadata table (sample_name, comment)

## Outputs

- MGF file with clustered MS2 spectra
- GNPS Feature Quantification table (TXT)
- Adduct-connection supplementary table (comma-separated) for IIMN collapsed ion-identity edges
- Sample metadata table for GNPS upload
- Annotated GNPS GraphML network with SIRIUS/CSI:FingerID annotations

## How to apply

Apply the OpenMS FileFilter to the consensus feature map to retain only features that carry MS2 information, since GNPS clusters MS2 spectra rather than consuming raw maps. Run the OpenMS GNPSExport tool together with all mzML files: it clusters the MS2 information into a single MGF file. Convert the filtered consensus map into a GNPS Feature Quantification table (TXT) with pyOpenMS, then generate the comma-separated adduct-connection supplementary table used for IIMN collapsed ion-identity edges and the sample metadata table derived from the dataset.tsv table. After the GNPS job finishes and the resulting network is downloaded, parse the GNPS-generated GraphML file and integrate #1-ranked SIRIUS/CSI:FingerID structural and formula predictions into the network for MSI level 2/3 annotation inspection.

## Related tools

- **OpenMS GNPSExport** (Clusters MS2 information from the filtered consensus feature map into a single MGF file for GNPS) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **OpenMS FileFilter** (Filters the consensus feature map to retain only features that carry MS2 information before export)
- **pyOpenMS** (Converts the filtered consensus map into the GNPS Feature Quantification table (TXT) and generates the adduct-connection supplementary table) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **GNPS** (Consumes the generated file set to run FBMN or IIMN jobs and produce the GraphML network)
- **SIRIUS** (Provides #1-ranked molecular formula and structural predictions that are integrated into the GNPS GraphML network)

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- All GNPS-required files (MGF, Feature Quantification TXT, adduct-connection table, metadata table) are present in upload-ready format.
- The MGF contains a single consolidated spectrum block per feature with MS2 information (GNPSExport clustering succeeded).
- The feature quantification table rows correspond to consensus features that survived the MS2-only FileFilter step.
- The downloaded GNPS GraphML network contains SIRIUS/CSI:FingerID annotations (MSI level 2/3) after integration.
- Adduct-connection table columns align with the adduct annotations in the consensus map so IIMN can collapse ion-identity edges.

## Limitations

- MSI level 3 annotations in the GraphML depend on #1-ranked SIRIUS and CSI:FingerID predictions; lower-ranked or failed predictions leave features unannotated.
- Spectral matching against a library is a separate step (score above 60, MSI level 2); this export only enables the network job, not library annotation.
- Vendor files other than Thermo must be converted independently using proteowizard before this step; Agilent/Bruker files skip the built-in file conversion rule.

## Evidence

- [methods] The GNPSExport tool (run together with all mzML files) clusters the MS2 information into a single MGF file, converts the consensus file into a Feature Quantification table (TXT), and generates a comma-separated supplementary table.: [evidence span withheld — anchor status: not_found]
- [readme] generate all the files necessary to create a GNPS FBMN or IIMN job at GNPS: "generate all the files necessary to create a GNPS [FBMN](https://ccms-ucsd.github.io/GNPSDocumentation/featurebasedmolecularnetworking-with-openms/) or"
- [readme] Integrate SIRIUS predictions to the network (GraphML) and GNPS library annotations to the feature matrix: "**Graph view**: Integrate SIRIUS predictions to the network (GraphML) and GNPS library annotations to the feature matrix - MSI level 2 (optional)."
- [readme] If you have Agilent, Bruker, or other vendor files, skip that step (write "FALSE" for rule file conversion in the config.yaml file: "If you have Agilent, Bruker, or other vendor files, skip that step (write "FALSE" for rule file conversion in the config.yaml file"
