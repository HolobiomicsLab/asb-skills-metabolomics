---
name: nmr-reference-spectrum-retrieval-from-hmdb
description: Use when building simulated 1H NMR case-vs-control spectra from known
  metabolites, or when detected peak ppm values must be queried against a reference
  metabolite database such as HMDB.
when_to_use_negative:
- Input is already a feature matrix or processed peak table — no reference retrieval
  is needed
- The analysis targets experimental spectra where no known-metabolite ground truth
  is required and HMDB lookup would add no annotation value
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0593
tools:
- name: HMDB
  role: Reference metabolite database queried with ppm values of peaks of interest
    to obtain reference 1H NMR spectra (e.g. for HMDB0000440 and HMDB0001336)
- name: speaq 2.0
  role: 'Downstream consumer: its peak-picking output (ppm values of actual peaks)
    is the query payload against HMDB; reference spectra feed the simulated benchmark'
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R implementation in which the retrieved reference spectra are combined into
    the simulated case-vs-control dataset
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:20c2e0c768b04fe5880ad0b272dd1092c2435024193e43eda8275c96d91b19f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - CluPA
  - HMDB
  - R
  - icoshift
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nmr-reference-spectrum-retrieval-from-hmdb

## Summary

Retrieves reference 1H NMR spectra (ppm peak positions) for known metabolites from the HMDB database, e.g. HMDB0000440 and HMDB0001336, to serve as ground-truth building blocks for simulated NMR spectra or to annotate detected peaks.

## When to use

Use when building simulated 1H NMR case-vs-control spectra from known metabolites, or when detected peak ppm values must be queried against a reference metabolite database such as HMDB.

## When NOT to use

- Input is already a feature matrix or processed peak table — no reference retrieval is needed
- The analysis targets experimental spectra where no known-metabolite ground truth is required and HMDB lookup would add no annotation value

## Inputs

- HMDB metabolite accession numbers (e.g. HMDB0000440, HMDB0001336)
- Target metabolite identities for the simulated mixture or annotation query

## Outputs

- Reference 1H NMR spectra (ppm values of actual peaks) for each queried metabolite
- Ground-truth peak lists used to construct simulated spectra or assign peak identities

## How to apply

First obtain the HMDB identifiers for the metabolites of interest (in the speaq 2.0 simulation benchmark, HMDB0000440 and HMDB0001336). Retrieve each metabolite's reference 1H NMR spectrum (the ppm values of its actual peaks) from the HMDB database. These reference spectra are then combined — with added noise, baseline distortion, and random peak shifts as prescribed in the study's S2 Appendix — to synthesize simulated spectra with known case-vs-control ground truth. The same retrieval pattern supports annotation: speaq 2.0 peak picking yields ppm values of actual peaks of interest that can be used to query HMDB. Validate retrieval by confirming that the reference peak lists correspond to the queried HMDB accession numbers before using them in downstream simulation or annotation.

## Related tools

- **HMDB** (Reference metabolite database queried with ppm values of peaks of interest to obtain reference 1H NMR spectra (e.g. for HMDB0000440 and HMDB0001336))
- **speaq 2.0** (Downstream consumer: its peak-picking output (ppm values of actual peaks) is the query payload against HMDB; reference spectra feed the simulated benchmark) — https://github.com/beirnaert/speaq
- **speaq R package** (R implementation in which the retrieved reference spectra are combined into the simulated case-vs-control dataset) — https://cran.r-project.org/package=speaq

## Examples

```
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)  # then query HMDB with peaks' ppm values; reference spectra for HMDB0000440/HMDB0001336 feed the simulation
```

## Evaluation signals

- Retrieved reference peak lists match the ppm peak patterns published for the queried HMDB accessions (HMDB0000440, HMDB0001336)
- Simulated 20-spectrum dataset (10 case, 10 control) built from the references reproduces case-vs-control separation under ROC and Precision-Recall evaluation
- Each retrieved spectrum's ppm axis covers the expected 1H NMR chemical-shift range for the metabolite
- Annotation queries against HMDB return the correct metabolite for the known peaks in the benchmark

## Limitations

- Severely shifted spectra reduce downstream match quality: 'The results of spectral alignment algorithms is not always optimal when dealing with severely shifted spectra' — retrieved references are only as useful as the peak positions recovered from the sample spectra
- HMDB reference spectra presuppose known metabolites; unknown or unanticipated metabolites cannot be simulated or annotated this way

## Evidence

- [intro] HMDB as peak-query database: "the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB [24]"
- [supplementary] simulated data benchmark uses HMDB-based spectra: "Simulated data based comparison of speaq 2.0 vs alignment algorithms."
- [abstract] speaq 2.0 avoids binning via peak picking then grouping: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [readme] README minimal example showing peak detection feeding downstream use: [evidence span withheld — anchor status: not_found]
