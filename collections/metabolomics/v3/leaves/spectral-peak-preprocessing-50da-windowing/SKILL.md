---
name: spectral-peak-preprocessing-50da-windowing
description: Use when preparing centroided LC-MS/MS spectra (MGF, mzXML, mzML or mzData)
  as input to DEREPLICATOR, VarQuest, DEREPLICATOR+ or MetaMiner database-search pipelines,
  or when reproducing MSM counts that require the paper's exact spectral preprocessing.
when_to_use_negative:
- The spectra are already peak-filtered or you plan to use a different peak-picking
  scheme — deviating from the six-peaks-per-50-Da rule will make P-values and target-decoy
  FDR estimates incomparable with the published results
- The downstream tool is not an NPDtools/GNPS Mohimani-lab pipeline (e.g. general
  metabolomics feature detection) — this filter is specific to the fragmentation-graph
  scoring used by DEREPLICATOR+
- The input data are profile-mode spectra that still need centroiding — run msconvert
  centroiding first, as the filter operates on discrete centroided peaks
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Performs the six-top-peaks-per-50-Da windowed filtering as its built-in spectral
    preprocessing before constructing fragmentation graphs and scoring MSMs
  repo: https://github.com/ablab/npdtools
- name: msconvert (ProteoWizard)
  role: Converts raw/vendor spectra formats to centroided MGF prior to the peak-filtering
    step
  repo: http://proteowizard.sourceforge.net/tools/msconvert.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:c51fbc1becde6a64b91a7d32044ce04878a37a3a7f4758f30bda3940a0c36ae8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR+
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spectral-peak-preprocessing-50da-windowing

## Summary

Preprocess centroided tandem mass spectra for DEREPLICATOR+/DEREPLICATOR database search by retaining only the six top-intensity peaks in each 50 Da window, reducing noise while preserving fragment ions needed for metabolite-spectrum matching.

## When to use

Use when preparing centroided LC-MS/MS spectra (MGF, mzXML, mzML or mzData) as input to DEREPLICATOR, VarQuest, DEREPLICATOR+ or MetaMiner database-search pipelines, or when reproducing MSM counts that require the paper's exact spectral preprocessing.

## When NOT to use

- The spectra are already peak-filtered or you plan to use a different peak-picking scheme — deviating from the six-peaks-per-50-Da rule will make P-values and target-decoy FDR estimates incomparable with the published results
- The downstream tool is not an NPDtools/GNPS Mohimani-lab pipeline (e.g. general metabolomics feature detection) — this filter is specific to the fragmentation-graph scoring used by DEREPLICATOR+
- The input data are profile-mode spectra that still need centroiding — run msconvert centroiding first, as the filter operates on discrete centroided peaks

## Inputs

- Centroided LC-MS/MS tandem mass spectra in MGF, mzXML, mzML or mzData format (e.g. converted from MassIVE accessions MSV000078604 / MSV000078839 with msconvert)

## Outputs

- Filtered MGF spectra with only the six top-intensity peaks retained per 50 Da window, ready for DEREPLICATOR+/DEREPLICATOR database search

## How to apply

First convert vendor/raw spectra to centroided MGF using the msconvert utility from the ProteoWizard package, since NPDtools natively parses MGF (via libmgf) and mzXML/mzData (via RAMP). Then apply DEREPLICATOR+'s windowed peak filtering: slide a 50 Da window across each spectrum and keep only the six most intense peaks inside each window, discarding the rest. This normalization reduces dominant high-intensity noise peaks so that scoring — the number of unique spectrum masses annotating nodes of a metabolite fragmentation graph — is not biased by peak-density differences between spectra. Apply this step identically to every spectra file before running the database search against the natural product structure database, because changing the filter changes P-values, FDR estimates, and the resulting compound/MSM counts.

## Related tools

- **DEREPLICATOR+** (Performs the six-top-peaks-per-50-Da windowed filtering as its built-in spectral preprocessing before constructing fragmentation graphs and scoring MSMs) — https://github.com/ablab/npdtools
- **msconvert (ProteoWizard)** (Converts raw/vendor spectra formats to centroided MGF prior to the peak-filtering step) — http://proteowizard.sourceforge.net/tools/msconvert.html

## Examples

```
msconvert spectra.raw --mgf --centroid --outdir mgf/ && python dereplicator+.py --spectra mgf/ --database npd_database --output dereplicator_plus_spectraactiseq_results.tsv
```

## Evaluation signals

- Output MGF spectra contain no more than 6 peaks within any 50 Da mass window
- All spectra are centroided and parse cleanly with the NPDtools MGF parser (libmgf-based)
- Downstream DEREPLICATOR+ run on the filtered spectra reproduces published counts on SpectraActiSeq: 488 compounds (8194 MSMs) at 1% FDR and 154 compounds (2666 MSMs) at 0% FDR
- Total peak count per spectrum decreases substantially relative to the unfiltered input while precursor masses are preserved

## Limitations

- The fixed six-peaks-per-50-Da rule may discard genuine low-intensity fragment ions, contributing to the high false-negative rate of the shared-peaks scoring scheme
- Filtering must be applied uniformly across target and decoy searches; inconsistent preprocessing invalidates FDR estimation
- DEREPLICATOR+ preprocessing assumes LC-MS/MS data in one of the supported open formats; other formats must be converted with msconvert first

## Evidence

- [discussion] Six-top-peaks-per-50-Da window filter is DEREPLICATOR+'s built-in spectral preprocessing: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [readme] msconvert is used to convert other formats to MGF before NPDtools pipelines: [evidence span withheld — anchor status: external_doc]
- [discussion] MSM scoring counts unique masses annotating fragmentation graph nodes, motivating uniform peak filtering: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [readme] NPDtools requires centroided spectra in open formats: [evidence span withheld — anchor status: external_doc]
