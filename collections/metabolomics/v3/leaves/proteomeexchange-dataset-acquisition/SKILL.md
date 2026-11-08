---
name: proteomeexchange-dataset-acquisition
description: Use when a protocol cites a ProteomeXchange identifier (e.g. PXD034932)
  or OSF/PRIDE link as the data source and the tandem mass spectra must be downloaded
  as MGF files before any spectral comparison.
when_to_use_negative:
- The required MGF peak lists are already available locally — downloading again wastes
  bandwidth and risks version mismatch
- The analysis goal is peptide/protein identification by database search rather than
  direct dataset-vs-dataset spectral comparison; compareMS2-class tools and their
  input data are not designed for that
- The cited deposit lacks MGF-format peak lists (e.g. only raw instrument files),
  which compareMS2 cannot consume directly
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Downstream consumer of the retrieved MGF tandem mass spectra; the deposit
    PXD034932 is its designated test data
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:b33dc11cee3b7f9089c312788989f55bd61a7d06ffede6ab2629435723c36a65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# proteomeexchange-dataset-acquisition

## Summary

Retrieves tandem mass spectrometry datasets from the ProteomeXchange/PRIDE archive by dataset identifier (e.g. PXD034932, the compareMS2 2.0 test data) so that MGF peak-list files are available locally for downstream pairwise comparison.

## When to use

Use when a protocol cites a ProteomeXchange identifier (e.g. PXD034932) or OSF/PRIDE link as the data source and the tandem mass spectra must be downloaded as MGF files before any spectral comparison.

## When NOT to use

- The required MGF peak lists are already available locally — downloading again wastes bandwidth and risks version mismatch
- The analysis goal is peptide/protein identification by database search rather than direct dataset-vs-dataset spectral comparison; compareMS2-class tools and their input data are not designed for that
- The cited deposit lacks MGF-format peak lists (e.g. only raw instrument files), which compareMS2 cannot consume directly

## Inputs

- ProteomeXchange dataset identifier (e.g. PXD034932) or PRIDE project URL
- Optional OSF repository link to supplementary sample datasets (e.g. primate sera, https://osf.io/sg796/)
- Local output directory for downloaded files

## Outputs

- MGF files of tandem mass spectra, one per LC-MS/MS dataset
- A local record of the source identifier (PXD accession) and downloaded file list for provenance

## How to apply

Resolve the dataset identifier against the PRIDE archive project page (e.g. https://www.ebi.ac.uk/pride/archive/projects/PXD034932, the compareMS2 2.0 test deposit) and download the deposited peak-list files, selecting the tandem MS files in MGF format that the downstream tool (compareMS2) consumes. Prefer the curated test material named by the authors: the PXD034932 deposit and the primate sera datasets hosted on OSF (https://osf.io/sg796/), which contain roughly 1,000 tandem mass spectra each and serve as a good test set. If reproducing the paper's five serum datasets, expect each to contain between 42,629 and 47,626 tandem mass spectra — use these counts as a sanity check on complete downloads. Verify each downloaded MGF parses as a peak list (query blocks with precursor m/z and fragment peaks) before handing it to the similarity computation.

## Related tools

- **compareMS2** (Downstream consumer of the retrieved MGF tandem mass spectra; the deposit PXD034932 is its designated test data) — https://github.com/524D/compareMS2.git

## Examples

```
curl -L "https://www.ebi.ac.uk/pride/archive/projects/PXD034932" -o pxd034932_project.html
```

## Evaluation signals

- The PRIDE project page for the identifier resolves (HTTP 200) and lists deposited files including MGF peak lists
- Each downloaded MGF file parses into query blocks containing precursor m/z and fragment peak lists without truncation errors
- For the paper's serum datasets, the number of tandem mass spectra per file falls in the reported 42,629–47,626 range
- Test datasets named by the authors (PXD034932, primate sera on OSF sg796) are retrievable and load successfully in compareMS2

## Limitations

- The extremes of none or all spectra being similar between two LC-MS/MS datasets are unlikely in real deposits, so test data should be checked for spectral content before interpreting downstream distances
- Full datasets of tens of thousands of spectra per file are large; compareMS2 pairwise comparisons took 40 minutes for 10 comparisons with default parameters, so plan retrieval and compute accordingly
- Dataset identifiers resolve to whatever the depositors uploaded; file formats other than MGF (e.g. raw files) require conversion not covered by this skill

## Evidence

- [abstract] The data have been deposited to ProteomeXchange with dataset identifier PXD034932.: "The data have been deposited to ProteomeXchange with dataset identifier PXD034932."
- [readme] See PRIDE Project PXD034932 for additional compareMS2 test data.: [evidence span withheld — anchor status: not_found]
- [readme] Phylogenetic tree based on sample primate sera datasets of 1,000 tandem mass spectra, as displayed during a compareMS2 run. This is a good test dataset for compareMS2.: [evidence span withheld — anchor status: external_doc]
- [results] five serum datasets contain between 42,629 and 47,626 tandem mass spectra each: "each containing between 42,629 and 47,626 tandem mass spectra"
