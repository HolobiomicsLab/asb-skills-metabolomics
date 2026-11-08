---
name: poisson-binomial-pvalue-computation
description: Use when scored MSMs from DEREPLICATOR+ spectral annotation (shared-peak
  scores against target and decoy fragmentation graphs) must be converted into P-values
  and FDR-estimated identifications before reporting significant matches.
when_to_use_negative:
- Input spectra or compound classes fall outside DEREPLICATOR+'s scope — like all
  existing metabolomics tools it is not applicable to all classes of compounds
- The fragmentation graph is non-tree-like and an unbiased (e.g. MCMC) probability
  estimate is required — the implemented recursion is only approximate in that case
- The analysis requires a scoring scheme beyond simple shared peaks, which has a high
  false-negative rate and the improved scoring is not yet implemented
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: DEREPLICATOR+
  role: Generates decoy fragmentation graphs for each target, scores MSMs, and computes
    Poisson-binomial P-values and FDR estimates for each match
  repo: https://github.com/ablab/npdtools
- name: GNPS
  role: Web platform through which the DEREPLICATOR+ pipeline, including P-value/FDR
    evaluation, can be run against molecular networking spectra
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:38c8fbcbbbf2bd4ccc0d3372e0ec768bbaafae2f5f2057c977032ea6439f3eac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR+
  - GNPS
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# poisson-binomial-pvalue-computation

## Summary

Computes the statistical significance of metabolite-spectrum matches (MSMs) produced by DEREPLICATOR+ using a Poisson-binomial model over generated decoy fragmentation graphs, and estimates the False Discovery Rate (FDR) at each P-value level so surviving compound-spectrum matches can be reported.

## When to use

Use when scored MSMs from DEREPLICATOR+ spectral annotation (shared-peak scores against target and decoy fragmentation graphs) must be converted into P-values and FDR-estimated identifications before reporting significant matches.

## When NOT to use

- Input spectra or compound classes fall outside DEREPLICATOR+'s scope — like all existing metabolomics tools it is not applicable to all classes of compounds
- The fragmentation graph is non-tree-like and an unbiased (e.g. MCMC) probability estimate is required — the implemented recursion is only approximate in that case
- The analysis requires a scoring scheme beyond simple shared peaks, which has a high false-negative rate and the improved scoring is not yet implemented

## Inputs

- MSM scores score(M,S) — number of unique spectrum masses annotating nodes in the target metabolite fragmentation graph
- Scores of matches against generated decoy fragmentation graphs
- Preprocessed centroided MGF/mzXML/mzML/mzData spectra (six top-intensity peaks per 50 Da window)

## Outputs

- P-value for each metabolite-spectrum match under the Poisson-binomial model
- FDR estimates at each P-value level
- significant_matches.tsv with SpecFile, Scan, Score, P-Value, FDR, and compound annotation columns

## How to apply

After DEREPLICATOR+ annotates target and decoy fragmentation graphs with the preprocessed spectrum (six top-intensity peaks per 50 Da window) and computes score(M,S) as the number of unique spectrum masses annotating a node in the fragmentation graph, evaluate each MSM's P-value under the Poisson-binomial model built from the decoy fragmentation graphs generated for each target. Then estimate the False Discovery Rate at each P-value level to calibrate the score threshold. Only compound-spectrum matches that survive the FDR evaluation are written to significant_matches.tsv with SpecFile, Scan, Score, P-Value, and FDR columns. Note that the P-value recursion is exact only for tree-like fragmentation graphs; for non-tree graphs it is approximate, and MCMC is described as a possible unbiased alternative but was not implemented.

## Related tools

- **DEREPLICATOR+** (Generates decoy fragmentation graphs for each target, scores MSMs, and computes Poisson-binomial P-values and FDR estimates for each match) — https://github.com/ablab/npdtools
- **GNPS** (Web platform through which the DEREPLICATOR+ pipeline, including P-value/FDR evaluation, can be run against molecular networking spectra) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp

## Evaluation signals

- Every reported match in significant_matches.tsv carries a Score, P-Value, and FDR entry
- A decoy fragmentation graph exists for each target fragmentation graph, so decoy score distributions are available for every scored target
- The FDR estimates are monotone with P-value level and consistent with the decoy-derived null distribution
- The module reproduces reported depth-annotation counts for validation compounds such as heterocyst glycolipid and chalcomycin before P-value computation

## Limitations

- P-value recursion is only approximate for non-tree fragmentation graphs; unbiased Markov Chain Monte Carlo estimation is described as possible but not implemented
- Simple shared peaks scoring results in a high false-negative rate, which propagates into P-value/FDR-based identifications; improved scoring not yet implemented
- DEREPLICATOR+ is not applicable to all classes of compounds

## Evidence

- [discussion] Computing The Statistical Significance Of Msms And Evaluating Fdr: "Computing The Statistical Significance Of Msms And Evaluating Fdr"
- [discussion] Decoy graph generation: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [discussion] Score definition: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] Spectrum preprocessing: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] MCMC not implemented: "For unbiased probability estimation Markov Chain Monte Carlo methods can be utilized."
- [readme] Tool availability: [evidence span withheld — anchor status: not_found]
