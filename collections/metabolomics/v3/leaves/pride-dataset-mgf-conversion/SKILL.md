---
name: pride-dataset-mgf-conversion
description: Use when raw files from a PRIDE accession such as PXD000561 must be fetched
  and converted from vendor format to MGF (ThermoRawFileParser) before spectral clustering
  or CPU/GPU clustering benchmarks.
when_to_use_negative:
- The spectra are already in MGF format — pass the MGF folder directly as input_filepath
  instead of re-downloading and reconverting raw files
- You only need spectrum identifications rather than raw spectra clustering input
  — these can be downloaded from MassIVE reanalysis RMSV000000091.3
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3292
tools:
- name: PRIDE
  role: Public proteomics archive from which the raw files for data set PXD000561
    are downloaded
- name: ThermoRawFileParser
  role: Converts the downloaded vendor raw files into MGF peak lists while retaining
    precursor charge 2 and charge 3 spectra for downstream clustering
- name: HyperSpec
  role: Downstream consumer of the converted MGF folder (input_filepath) that runs
    the five-stage clustering pipeline and exports results as CSV
  repo: https://github.com/wh-xu/Hyper-Spec
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:cf64cacb3d8f4bd9f0af70f6435e5ee967b8ada49129a90f4e9ffaaa63bf4c57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - MS-Cluster
  - MaRaCluster
  - PRIDE
  - Python
  - ThermoRawFileParser
  - graphics processing units
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pride-dataset-mgf-conversion

## Summary

Retrieves raw MS files for a PRIDE accession (e.g., PXD000561, the 131 GB draft human proteome data set) and converts vendor raw files to MGF peak lists with ThermoRawFileParser so clustering tools such as HyperSpec, MS-Cluster, spectra-cluster, and MaRaCluster can consume them.

## When to use

Use when raw files from a PRIDE accession such as PXD000561 must be fetched and converted from vendor format to MGF (ThermoRawFileParser) before spectral clustering or CPU/GPU clustering benchmarks.

## When NOT to use

- The spectra are already in MGF format — pass the MGF folder directly as input_filepath instead of re-downloading and reconverting raw files
- You only need spectrum identifications rather than raw spectra clustering input — these can be downloaded from MassIVE reanalysis RMSV000000091.3

## Inputs

- Raw vendor mass spectrometry files for PRIDE accession PXD000561 (draft human proteome data set, 25 million MS/MS spectra, 131 GB of MS data)
- PRIDE database accession/download access for the target proteomics data set

## Outputs

- Folder of MGF-format peak files with precursor charge 2 and 3 spectra retained, usable directly as HyperSpec input_filepath or as input for MS-Cluster, spectra-cluster, and MaRaCluster

## How to apply

Download the raw files for the chosen PRIDE accession — in the HyperSpec paper the largest data set, PXD000561 (draft human proteome, 25 million MS/MS spectra, 131 GB), is used for runtime and clustering quality evaluation. Convert the vendor raw files to MGF peak lists with ThermoRawFileParser, retaining precursor charge 2 and charge 3 spectra to match HyperSpec's default --cluster_charges 2 3 configuration. Collect the MGF files in a single folder that serves as the input_filepath for HyperSpec (python src/main.py <mgf_folder> output.csv) or as the shared input for the CPU baselines MS-Cluster, spectra-cluster, and MaRaCluster. Rationale: peak information accounts for over 95% of raw-file data volume and MGF is the interchange format every benchmarked clustering tool accepts, so peak lists plus precursor m/z/charge metadata must survive conversion intact. Verify by re-parsing the MGF and later checking that HyperSpec's exported parquet metadata records bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, and is_representative for every spectrum.

## Related tools

- **PRIDE** (Public proteomics archive from which the raw files for data set PXD000561 are downloaded)
- **ThermoRawFileParser** (Converts the downloaded vendor raw files into MGF peak lists while retaining precursor charge 2 and charge 3 spectra for downstream clustering)
- **HyperSpec** (Downstream consumer of the converted MGF folder (input_filepath) that runs the five-stage clustering pipeline and exports results as CSV) — https://github.com/wh-xu/Hyper-Spec

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg hc_complete --cluster_charges 2 3 --eps=0.25 --refine
```

## Evaluation signals

- Converted MGF set covers the expected scale: PXD000561 is 25 million MS/MS spectra from 131 GB of source MS data, and the HyperSpec run reported on over 21 million spectra
- Precursor charge 2 and 3 spectra are retained, consistent with HyperSpec's default --cluster_charges 2 3; downstream parquet metadata must record precursor_charge and precursor_mz per spectrum
- HyperSpec accepts the MGF folder directly as input_filepath and produces output.csv clustering results plus the parquet metadata file without parser errors
- The same converted MGF files must run through the CPU baselines (MS-Cluster, spectra-cluster, MaRaCluster) so stage-wise preprocessing runtime shares (20-90% of total runtime) are computed on identical input
- Measured end-to-end HyperSpec runtime on the converted data should be on the order of the reported ~24 min, indicating the conversion preserved the spectra needed for full-scale clustering

## Limitations

- Clustering the full PXD000561 MGF data set requires a GTX 3090 with larger GPU memory (GTX 1080Ti suffices only for smaller-scale data sets), so successful conversion alone does not guarantee the downstream run completes
- HyperSpec has only been tested on Linux with NVIDIA GTX 1080Ti and GTX 3090 GPUs; other hardware is untested according to the README
- Only charge 2 and 3 spectra are clustered by default; spectra with other precursor charges present in the MGF are excluded from clustering unless --cluster_charges is changed

## Evidence

- [methods] The largest data set, PXD000561,1 was used for runtime and clustering quality evaluation.: "The largest data set, PXD000561,1 was used for runtime and clustering quality evaluation."
- [methods] For all data sets, raw files were downloaded from PRIDE: "For all data sets, raw files were downloaded from PRIDE"
- [intro] amounting to 25 million MS/MS spectra and 131 GB of MS data: "amounting to 25 million MS/MS spectra and 131 GB of MS data"
- [readme] HyperSpec supports running using the command line and takes `MGF` peak files as input and exports the clustering result as a CSV file: "HyperSpec supports running using the command line and takes `MGF` peak files as input and exports the clustering result as a CSV file"
- [readme] The charges to be clustered. (default: 2 3): "The charges to be clustered. (default: 2 3)"
