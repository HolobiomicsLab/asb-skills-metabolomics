---
name: mass-spectra-database-search
description: Use when centroided LC-MS/MS spectra (MGF, mzXML, mzML, or mzData) must
  be dereplicated against a known natural-product structure database (e.g., DNP +
  GNPS structures) with FDR-controlled reporting of metabolite–spectrum matches.
when_to_use_negative:
- Input spectra are of unknown/novel metabolites not represented in the structure
  database — the tool dereplicates known structures only
- Compounds outside the metabolite classes DEREPLICATOR+ covers (the tool 'is not
  applicable to all the classes of compounds'); for peptides-only workflows the original
  DEREPLICATOR baseline may be preferred for comparison
- Spectra are profile-mode and uncentroided — the pipelines require centroided data
  in an open format
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: 'Primary search engine: scores spectra against metabolite fragmentation graphs
    and generates decoy fragmentation graphs for target-decoy FDR estimation'
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Baseline tool limited to peptidic natural products (PNPs), run in parallel
    for comparison of identification yields
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR-G
  role: Variant of DEREPLICATOR (all C–N bonds cut rather than amide bonds only) run
    as a comparison baseline on the same spectra
- name: GNPS
  role: Web platform hosting the DEREPLICATOR+ workflow and the spectral repository
    / molecular networking infrastructure searched
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: ProteoWizard msconvert
  role: Converts raw/other vendor spectra formats to centroided MGF before database
    search
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:a156d34a641d6071ed80ab00b7bcc3ff670fd270d600aa0228c237513733ee4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - GNPS
  - ProteoWizard msconvert
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mass-spectra-database-search

## Summary

Dereplication of tandem mass spectra by database search against known natural-product structures using DEREPLICATOR+ fragmentation-graph scoring, with target-decoy FDR estimation. This enables identification of metabolite–spectrum matches (MSMs) across polyketides, terpenes, alkaloids, and peptidic natural products in large LC-MS/MS datasets.

## When to use

Use when centroided LC-MS/MS spectra (MGF, mzXML, mzML, or mzData) must be dereplicated against a known natural-product structure database (e.g., DNP + GNPS structures) with FDR-controlled reporting of metabolite–spectrum matches.

## When NOT to use

- Input spectra are of unknown/novel metabolites not represented in the structure database — the tool dereplicates known structures only
- Compounds outside the metabolite classes DEREPLICATOR+ covers (the tool 'is not applicable to all the classes of compounds'); for peptides-only workflows the original DEREPLICATOR baseline may be preferred for comparison
- Spectra are profile-mode and uncentroided — the pipelines require centroided data in an open format

## Inputs

- Centroided LC-MS/MS spectra files in MGF, mzXML, mzML, or mzData format
- Chemical structure database of known natural products (e.g., Dictionary of Natural Products combined with GNPS library structures)
- Genome nucleotide sequences or genome-mining tool output (only for metabologenomic pipelines such as MetaMiner)

## Outputs

- List of metabolite–spectrum matches (MSMs) with scores, P-values, and FDR annotations (e.g., dereplicator_plus_gnps_search_results.tsv with tool, number of MSMs, distinct compounds)
- Top-rank identification counts and percentages per tool and precursor mass tolerance (e.g., top_rank_percentages.csv)

## How to apply

Convert raw spectra to a centroided open format (MGF via ProteoWizard msconvert, or mzXML/mzML/mzData which NPDtools parses natively). Preprocess each spectrum by retaining only the six top-intensity peaks in each 50 Da window. Search spectra against the chemical structure database (e.g., 86,665 structures from the Dictionary of Natural Products merged with GNPS library structures) by scoring spectra against metabolite fragmentation graphs, where score(M,S) is the number of unique masses in S that annotate a node in the fragmentation graph of M. Generate a decoy fragmentation graph for each target to compute P-values and estimate FDR, then filter at 0%, 1%, or 3% FDR (the DEREPLICATOR+ score threshold for 1% FDR is 12). Optionally sweep precursor mass tolerance (e.g., 0.5 Da vs 0.02 Da) and rank candidate metabolite–spectrum matches to report top-rank identification rates.

## Related tools

- **DEREPLICATOR+** (Primary search engine: scores spectra against metabolite fragmentation graphs and generates decoy fragmentation graphs for target-decoy FDR estimation) — https://github.com/ablab/npdtools
- **DEREPLICATOR** (Baseline tool limited to peptidic natural products (PNPs), run in parallel for comparison of identification yields) — https://github.com/ablab/npdtools
- **DEREPLICATOR-G** (Variant of DEREPLICATOR (all C–N bonds cut rather than amide bonds only) run as a comparison baseline on the same spectra)
- **GNPS** (Web platform hosting the DEREPLICATOR+ workflow and the spectral repository / molecular networking infrastructure searched) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **ProteoWizard msconvert** (Converts raw/other vendor spectra formats to centroided MGF before database search)

## Examples

```
msconvert raw_spectra.raw --mgf --filter peakpicking && bin/dereplicator+.py --spectra raw_spectra.mgf --database dnp_structures
```

## Evaluation signals

- Benchmark top-rank rates must match reported values on 5473 SpectraLibrary spectra vs 86,665 structures: DEREPLICATOR+ 34% (1878) at 0.5 Da and 44% (2414) at 0.02 Da; DEREPLICATOR-G 30%/40%; DEREPLICATOR 17%/25%
- SpectraGNPS full-dataset search at 1% FDR should yield ~410,539 MSMs / 5,336 compounds for DEREPLICATOR+, vs 82,260 MSMs / 1,417 compounds (DEREPLICATOR-G) and 41,403 MSMs / 974 compounds (DEREPLICATOR)
- DEREPLICATOR+ identifications should be roughly an order of magnitude more natural products than the DEREPLICATOR and DEREPLICATOR-G baselines
- SpectraActiSeq (MSV000078604, MSV000078839) validation: 488 compounds (8194 MSMs) at 1% FDR, 154 compounds (2666 MSMs) at 0% FDR, 1024 compounds at 3% FDR
- Every reported MSM must pass the 1% FDR filter, corresponding to a DEREPLICATOR+ score threshold of 12

## Limitations

- Simple shared-peaks scoring results in a high false-negative rate; an improved scoring scheme was not implemented
- P-value computation relies on a recursion that is only approximate for non-tree (cyclic) fragmentation graphs; unbiased MCMC estimation is described as possible but not implemented
- Similar to all existing metabolomics tools, DEREPLICATOR+ is not applicable to all classes of compounds
- No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS, complicating exact reproduction

## Evidence

- [readme] Spectra files must be centroided and be in an open spectrum format (MGF, mzXML, mzML or mzData).: [evidence span withheld — anchor status: external_doc]
- [discussion] Score is unique matched masses annotating fragmentation-graph nodes.: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] Six top peaks per 50 Da window.: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] Decoy fragmentation graphs for FDR estimation.: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [methods] msconvert is used to produce MGF.: "uses msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF"
- [discussion] Simple shared peaks scoring causes high false negatives.: [evidence span withheld — anchor status: not_found]
