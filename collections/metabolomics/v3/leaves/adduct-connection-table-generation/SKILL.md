---
name: adduct-connection-table-generation
description: 'Use when preparing files for a GNPS IIMN job with collapsed ion-identity
  edges: a filtered consensus feature map with adduct annotations must be converted
  into the adduct-connection supplementary table expected by GNPS.'
when_to_use_negative:
- When running only a plain GNPS Feature-Based Molecular Networking (FBMN) job without
  collapsed ion-identity edges — the adduct-connection table is specific to IIMN.
- When the consensus feature map has not yet undergone OpenMS adduct annotation and
  feature linking, so no ion-identity/adduct relationships exist to tabulate.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: UmetaFlow
  role: Workflow that implements the GNPS-export sub-workflow, including the adduct-connection
    supplementary table generation via pyOpenMS
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: OpenMS GNPSExport
  role: Clusters MS2 information into a single MGF file and, together with the FileFilter
    step, feeds the consensus map used for the adduct-connection table
- name: pyOpenMS
  role: Python bindings used to convert the filtered consensus feature map into the
    adduct-connection supplementary table and the GNPS Feature Quantification table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:edf465ec58740e373495210e72ea37748240bc74e3531b91e26c9110627c8330
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - OpenMS
  - OpenMS GNPSExport
  - Python
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Adduct connection table generation

## Summary

Generates the comma-separated supplementary table that describes ion-identity/adduct relationships between features, enabling GNPS Ion Identity Molecular Networking (IIMN) to collapse adduct edges.

## When to use

Use when preparing files for a GNPS IIMN job with collapsed ion-identity edges: a filtered consensus feature map with adduct annotations must be converted into the adduct-connection supplementary table expected by GNPS.

## When NOT to use

- When running only a plain GNPS Feature-Based Molecular Networking (FBMN) job without collapsed ion-identity edges — the adduct-connection table is specific to IIMN.
- When the consensus feature map has not yet undergone OpenMS adduct annotation and feature linking, so no ion-identity/adduct relationships exist to tabulate.

## Inputs

- OpenMS consensus feature map (consensusXML) filtered to features carrying MS2 information, with adduct annotations
- dataset.tsv sample metadata used by the UmetaFlow configuration

## Outputs

- Comma-separated adduct-connection supplementary table for GNPS IIMN
- Set of GNPS upload-ready files (MGF, Feature Quantification table, adduct-connection table, metadata table)

## How to apply

After OpenMS adduct annotation and feature linking have produced a consensus feature map carrying MS2 information, apply OpenMS FileFilter to retain only features with MS2 spectra. Then use pyOpenMS in the UmetaFlow GNPS-export notebook (4_GNPSExport.ipynb) to convert this filtered consensus map into a comma-separated supplementary table describing the adduct-connection relationships between features. Alongside the GNPSExport-derived MGF and the GNPS Feature Quantification table (TXT), this supplementary table is one of the upload files for a GNPS IIMN job; in IIMN, the network is then rendered with collapsed ion-identity edges. Correctness is judged by checking that the table covers all adduct-linked feature groups and uploads successfully together with the MGF and quantification table.

## Related tools

- **UmetaFlow** (Workflow that implements the GNPS-export sub-workflow, including the adduct-connection supplementary table generation via pyOpenMS) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **OpenMS GNPSExport** (Clusters MS2 information into a single MGF file and, together with the FileFilter step, feeds the consensus map used for the adduct-connection table)
- **pyOpenMS** (Python bindings used to convert the filtered consensus feature map into the adduct-connection supplementary table and the GNPS Feature Quantification table)

## Examples

```
snakemake --use-conda results/gnps/adduct_connections.tsv --cores all
```

## Evaluation signals

- All adduct-linked feature groups present in the filtered consensus map are represented in the supplementary table (no missing adduct connections).
- The output file is comma-separated and in an upload-ready supplementary-table format accepted by GNPS IIMN.
- The table is generated together with the complete GNPS file set: MGF from GNPSExport, GNPS Feature Quantification table (TXT), and sample metadata table derived from dataset.tsv.
- The downloaded GNPS IIMN network (GraphML) renders with collapsed ion-identity edges, confirming the supplementary table was correctly consumed.

## Limitations

- The article does not state the exact software version numbers for UmetaFlow, OpenMS, or pyOpenMS, so reproducibility of the adduct-connection output depends on the user's installed OpenMS 3.x/pyOpenMS versions.
- Benchmark performance numbers for the overall GNPS-export step (Tables S5, S7) are only in supplementary files, not the main text.
- Only feature maps that carry MS2 information survive the FileFilter step; features without MS2 spectra cannot contribute adduct connections for IIMN.

## Evidence

- [other] The GNPSExport tool (run together with all mzML files) clusters the MS2 information into a single MGF file, converts the consensus file into a Feature Quantification table (TXT), and generates a comma-separated supplementary table.: [evidence span withheld — anchor status: not_found]
- [readme] generate all the files necessary to create a FBMN or IIMN job at GNPS: [evidence span withheld — anchor status: external_doc]
- [intro] using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [other] Apply OpenMS FileFilter to the consensus feature map to retain only features that carry MS2 information: [evidence span withheld — anchor status: not_found]
