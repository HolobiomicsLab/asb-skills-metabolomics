---
name: top-k-match-evaluation
description: Use when you need to quantify how accurately a database-search dereplication
  tool identifies metabolites — e.g., rank-1/top-3 recall and precision on spectra
  with known compound annotations at a fixed FDR cutoff such as 3%.
when_to_use_negative:
- The spectra lack known compound annotations, so recall/precision cannot be computed
  — only FDR-estimated identification rates are possible
- The tool of interest does not produce ranked candidate lists, making top-k precision/recall
  undefined
- The compound class under study is outside DEREPLICATOR+'s scope — the paper notes
  it 'is not applicable to all the classes of compounds'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Database search tool whose rank-1/top-3 identifications are evaluated; scored
    MSMs by number of unique matched masses, with decoy fragmentation graphs for FDR
    estimation
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Predecessor PNP-only dereplication tool included in the same NPDtools toolkit,
    used for comparison of identification rates
  repo: https://github.com/ablab/npdtools
- name: msconvert (ProteoWizard)
  role: Converts spectra in other formats to MGF before running the DEREPLICATOR+
    database search used in the evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:aa8eec12a59ee93f89f60ad7ff687aff3ff9375766a18cf47fdfbf6ad931cbd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# top-k-match-evaluation

## Summary

Computes rank-1 and top-3 recall and precision of a spectral dereplication pipeline (e.g., DEREPLICATOR+) against known compound annotations on a benchmark such as SpectraLibrary, at a fixed FDR threshold.

## When to use

Use when you need to quantify how accurately a database-search dereplication tool identifies metabolites — e.g., rank-1/top-3 recall and precision on spectra with known compound annotations at a fixed FDR cutoff such as 3%.

## When NOT to use

- The spectra lack known compound annotations, so recall/precision cannot be computed — only FDR-estimated identification rates are possible
- The tool of interest does not produce ranked candidate lists, making top-k precision/recall undefined
- The compound class under study is outside DEREPLICATOR+'s scope — the paper notes it 'is not applicable to all the classes of compounds'

## Inputs

- Benchmark spectral dataset of compounds with known structures and reference tandem MS/MS spectra (e.g., SpectraLibrary; centroided MGF, mzXML, mzML, or mzData)
- Reference chemical structure database of known natural products
- DEREPLICATOR+ output: metabolite-spectrum matches (MSMs) with scores and candidate rankings

## Outputs

- Rank-1 recall (correct rank-1 identifications / total benchmark spectra)
- Rank-1 precision (correct rank-1 matches / total rank-1 identifications)
- Top-3 recall and top-3 precision (correct compound within top-3 candidates)
- Set of retained MSMs passing the applied FDR threshold

## How to apply

Load a benchmark spectral dataset of compounds with known structures and their reference tandem mass spectra (e.g., SpectraLibrary; spectra must be centroided, in MGF/mzXML/mzML/mzData, converted with ProteoWizard msconvert if needed). Run DEREPLICATOR+ to search each spectrum against a reference compound database, where score(M,S) is the number of unique masses in S annotating a node of the metabolite fragmentation graph of M, and significance is estimated by generating a decoy fragmentation graph for each target fragmentation graph. Apply the chosen FDR threshold (the benchmark uses 3%) to retain only significant metabolite-spectrum matches (MSMs). Against the known annotations, compute rank-1 recall as identified spectra / total spectra and rank-1 precision as correct rank-1 matches / total rank-1 matches; repeat counting the correct compound among the top-3 scoring candidates. Validate by comparing to the published reference: rank-1 recall 33% (1802/5473), rank-1 precision 58% (1802/3087), top-3 recall 48% (2614/5473), top-3 precision 85% (2614/3087).

## Related tools

- **DEREPLICATOR+** (Database search tool whose rank-1/top-3 identifications are evaluated; scored MSMs by number of unique matched masses, with decoy fragmentation graphs for FDR estimation) — https://github.com/ablab/npdtools
- **DEREPLICATOR** (Predecessor PNP-only dereplication tool included in the same NPDtools toolkit, used for comparison of identification rates) — https://github.com/ablab/npdtools
- **msconvert (ProteoWizard)** (Converts spectra in other formats to MGF before running the DEREPLICATOR+ database search used in the evaluation)

## Evaluation signals

- Rank-1 recall on SpectraLibrary at 3% FDR reproduces 33% (1802/5473 spectra)
- Rank-1 precision on SpectraLibrary at 3% FDR reproduces 58% (1802/3087 identifications)
- Top-3 recall and precision reproduce 48% (2614/5473) and 85% (2614/3087)
- Number of retained MSMs equals the number of identifications used as the precision denominator after applying the FDR threshold
- FDR is estimated from decoy matches: a decoy fragmentation graph is generated for each target fragmentation graph

## Limitations

- DEREPLICATOR+ uses a simple shared-peaks scoring scheme that results in a high false-negative rate
- The p-value recursion is only approximate for non-tree fragmentation graphs; unbiased estimation via MCMC is possible but not implemented
- DEREPLICATOR+ is not applicable to all classes of compounds
- No version numbers are stated for DEREPLICATOR+ or GNPS in the paper, which can complicate exact benchmark reproduction

## Evidence

- [discussion] Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M.: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph.: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [discussion] DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da.: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [readme] Dereplicator+ — a tool for identification of metabolites (both peptidic and non-peptidic) through database search of mass spectra: [evidence span withheld — anchor status: not_found]
