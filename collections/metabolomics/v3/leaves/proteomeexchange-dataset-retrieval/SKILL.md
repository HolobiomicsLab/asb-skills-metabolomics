---
name: proteomeexchange-dataset-retrieval
description: Use when a benchmark or reproduction task needs raw tandem mass spectra
  for a published study and the paper cites a ProteomeXchange/PRIDE identifier such
  as PXD034932, so the five serum LC-MS/MS datasets can be fetched before compareMS2.
when_to_use_negative:
- The task only needs processed peptide/protein identifications or a quantified feature
  table rather than raw tandem mass spectra
- No ProteomeXchange or PRIDE accession is cited, so the dataset cannot be resolved
  to a public archive
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Downstream consumer of the retrieved tandem mass spectra; the article designates
    the PXD034932 serum datasets as compareMS2 test data
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:2b5e81215af4af1ccadbf9c805f856aae055fdefec550208b88601a1deb0599f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - Intel Xeon W-2135
  - compareMS2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# proteomeexchange-dataset-retrieval

## Summary

Retrieves LC-MS/MS datasets identified by a ProteomeXchange accession (e.g. PXD034932) via the EBI PRIDE archive, providing the tandem mass spectra inputs (e.g. five serum datasets of 42,629–47,626 MS2 spectra) needed for downstream spectral-comparison analyses such as compareMS2 distance matrices.

## When to use

Use when a benchmark or reproduction task needs raw tandem mass spectra for a published study and the paper cites a ProteomeXchange/PRIDE identifier such as PXD034932, so the five serum LC-MS/MS datasets can be fetched before compareMS2.

## When NOT to use

- The task only needs processed peptide/protein identifications or a quantified feature table rather than raw tandem mass spectra
- No ProteomeXchange or PRIDE accession is cited, so the dataset cannot be resolved to a public archive

## Inputs

- ProteomeXchange/PRIDE accession identifier (e.g. PXD034932)
- EBI PRIDE archive project URL (e.g. https://www.ebi.ac.uk/pride/archive/projects/PXD034932)

## Outputs

- Downloaded raw LC-MS/MS files for five serum datasets (42,629–47,626 tandem mass spectra each), ready for conversion to MGF or direct compareMS2 ingestion

## How to apply

Resolve the accession (here PXD034932) against the PRIDE archive URL given in the article and README (ebi.ac.uk/pride/archive/projects/PXD034932), then download the five new serum LC-MS/MS datasets. Convert or inspect the vendor files to tandem mass spectra; compareMS2 operates on MS2 spectra (MGF in GUI mode), and the article specifies each serum dataset contains between 42,629 and 47,626 tandem mass spectra, which serves as a sanity check that all five datasets downloaded completely. These datasets are also flagged as compareMS2 test data, so the retrieved files can seed phylogenetic-tree, spectra2species, or heatmap runs.

## Related tools

- **compareMS2** (Downstream consumer of the retrieved tandem mass spectra; the article designates the PXD034932 serum datasets as compareMS2 test data) — https://github.com/524D/compareMS2.git

## Examples

```
wget -r -np https://www.ebi.ac.uk/pride/archive/projects/PXD034932 && comparems2 --cli --files serum/*.mgf
```

## Evaluation signals

- All five serum datasets download and each file yields a tandem mass spectrum count in the 42,629–47,626 range reported in the article
- Resolved dataset identifier matches PXD034932 as deposited to ProteomeXchange (dataset identifier stated in the abstract)
- Retrieved files can be compared directly with compareMS2 in phylogenetic-tree mode, consistent with the README noting PRIDE PXD034932 as compareMS2 test data

## Limitations

- Vendor-format raw files may require conversion (compareMS2 GUI mode works on MGF); the README does not specify a conversion command for these datasets
- Runtime and spectra counts depend on the machine; the article's 40-min benchmark was on an Intel Xeon W-2135 at 3.70 GHz, so replication across hardware varies

## Evidence

- [abstract] Dataset accession: "The data have been deposited to ProteomeXchange with dataset identifier PXD034932."
- [results] Spectra counts: "each containing between 42,629 and 47,626 tandem mass spectra"
- [readme] PRIDE link: [evidence span withheld — anchor status: external_doc]
