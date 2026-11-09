---
name: target-decoy-fdr-estimation
description: Use when dereplicating LC-MS/MS spectra with DEREPLICATOR/DEREPLICATOR+
  and reported MSM counts must be controlled at a chosen FDR level (0%, 1%, or 3%)
  rather than by raw score alone.
when_to_use_negative:
- Input spectra are not centroided tandem MS/MS data convertible to MGF/mzXML/mzML/mzData
  — the Poisson-binomial target-decoy machinery is defined only for DEREPLICATOR+
  MSM scoring
- The compound class is unsupported by the metabolite fragmentation graph model —
  the article notes DEREPLICATOR+ 'is not applicable to all the classes of compounds'
- A simple raw score cutoff suffices and no FDR-controlled identification counts are
  required
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DEREPLICATOR+
  role: Generates decoy fragmentation graphs for each target graph, computes MSM P-values
    under the Poisson-binomial model, and estimates FDR
  repo: github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Baseline peptidic natural product search using the same target-decoy strategy
    to compute MSM P-values and FDR
  repo: github.com/ablab/npdtools
- name: GNPS
  role: Web platform where the DEREPLICATOR+ workflow with target-decoy FDR estimation
    can be run online
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:9c2d2e54ba5b2cde2ee53fda3446dbf0d24d47a49a075b0d2af6d11d8282343f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - GNPS
  - ProteoWizard msconvert
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# target-decoy-fdr-estimation

## Summary

Estimate the statistical significance and False Discovery Rate of metabolite–spectrum matches (MSMs) in DEREPLICATOR+ by generating a decoy fragmentation graph for each target metabolite fragmentation graph, computing P-values under a Poisson-binomial model, and filtering matches at 0%, 1%, and 3% FDR thresholds.

## When to use

Use when dereplicating LC-MS/MS spectra with DEREPLICATOR/DEREPLICATOR+ and reported MSM counts must be controlled at a chosen FDR level (0%, 1%, or 3%) rather than by raw score alone.

## When NOT to use

- Input spectra are not centroided tandem MS/MS data convertible to MGF/mzXML/mzML/mzData — the Poisson-binomial target-decoy machinery is defined only for DEREPLICATOR+ MSM scoring
- The compound class is unsupported by the metabolite fragmentation graph model — the article notes DEREPLICATOR+ 'is not applicable to all the classes of compounds'
- A simple raw score cutoff suffices and no FDR-controlled identification counts are required

## Inputs

- Scored target metabolite–spectrum matches (MSMs) from DEREPLICATOR+ fragmentation-graph search
- Target fragmentation graphs built from metabolite structures by disconnecting C–N, C–O, and C–C bonds
- Preprocessed centroided spectra (MGF, mzXML, mzML, or mzData) with six top-intensity peaks per 50 Da window
- Generated decoy fragmentation graphs (one per target fragmentation graph)

## Outputs

- significant_matches.tsv with SpecFile, Scan, Score, P-Value, FDR, and compound annotation columns
- MSM and unique-compound counts at 0%, 1%, and 3% FDR thresholds (e.g., dereplicator_fdr_counts.tsv)

## How to apply

After scoring metabolite–spectrum matches as the number of unique spectrum masses annotating nodes of each metabolite's fragmentation graph (spectra preprocessed to retain six top-intensity peaks per 50 Da window), DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph. The P-value of each MSM is computed under a Poisson-binomial model over these decoy graphs, and the FDR is evaluated at each P-value level via the target-decoy strategy. Decision point: select an FDR threshold (the paper reproduces counts at 0%, 1%, and 3%); e.g., the SpectraGNPS search at 1% FDR corresponds to a score threshold of 12, yielding 410,539 MSMs and 5,336 compounds. Surviving matches are written with Score, P-Value, and FDR columns so downstream filtering and counting can be verified.

## Related tools

- **DEREPLICATOR+** (Generates decoy fragmentation graphs for each target graph, computes MSM P-values under the Poisson-binomial model, and estimates FDR) — github.com/ablab/npdtools
- **DEREPLICATOR** (Baseline peptidic natural product search using the same target-decoy strategy to compute MSM P-values and FDR) — github.com/ablab/npdtools
- **GNPS** (Web platform where the DEREPLICATOR+ workflow with target-decoy FDR estimation can be run online) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp

## Examples

```
dereplicator+.py --spectra spectraactiseq.mgf --database npd_structures --fdr-thresholds 0,1,3 --output dereplicator_plus_spectraactiseq_results.tsv
```

## Evaluation signals

- Reproduced SpectraActiSeq counts must equal 488 compounds (8194 MSMs) at 1% FDR, 154 compounds (2666 MSMs) at 0% FDR, and 1024 compounds at 3% FDR for DEREPLICATOR+
- DEREPLICATOR baseline on SpectraActiSeq must yield 73 compounds (166 MSMs) at 1% FDR and 66 compounds (148 MSMs) at 0% FDR
- Every reported match in the output TSV carries a Score, P-Value, and FDR entry
- At 1% FDR on SpectraGNPS, DEREPLICATOR+ identifies roughly an order of magnitude more compounds (5,336) than DEREPLICATOR (974) and DEREPLICATOR-G (1,417)
- Selecting 1% FDR on SpectraGNPS corresponds to a score threshold of 12

## Limitations

- The P-value recursion is only approximate for non-tree (cyclic) fragmentation graphs; unbiased MCMC estimation is described as possible but not implemented
- The simple shared-peaks scoring feeding the FDR estimation results in a high false-negative rate
- No version numbers are stated for DEREPLICATOR+ or DEREPLICATOR, complicating exact reproduction

## Evidence

- [discussion] DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph.: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [discussion] Computing the statistical significance of MSMs and evaluating the FDR: "Computing The Statistical Significance Of Msms And Evaluating Fdr"
- [methods] Spectra conversion and supported formats: "uses msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF"
- [readme] Spectra files must be centroided and in an open spectrum format: [evidence span withheld — anchor status: external_doc]
- [discussion] P-value approximation caveat for non-tree graphs: "For unbiased probability estimation Markov Chain Monte Carlo methods can be utilized."
