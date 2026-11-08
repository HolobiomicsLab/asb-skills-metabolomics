---
name: false-discovery-rate-estimation-decoy-search
description: Use when benchmarking DEREPLICATOR+ database search identifications (e.g.,
  on SpectraLibrary or GNPS spectra) and you need a target-decoy FDR estimate to define
  a significance cutoff for recall/precision measurement.
when_to_use_negative:
- The target compounds fall outside classes DEREPLICATOR+ can model — the article
  states 'Similar to all the existing metabolomics tools, DEREPLICATOR+ is not applicable
  to all the classes of compounds.'
- Spectra are already confidently identified against a small, fully validated reference
  library where decoy competition would not change the identification set.
- The goal is de novo or spectral-network variant propagation rather than FDR-controlled
  database search of known metabolites.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3474
tools:
- name: DEREPLICATOR+
  role: Performs the target-decoy database search of mass spectra against metabolite
    fragmentation graphs and computes MSM statistical significance and FDR
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Predecessor tool for peptidic natural product (PNP) dereplication included
    in NPDtools; relevant for PNP-only search pipelines
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:771b03ef22e010a614edae7a98ccf96677ffe5af6974684365603b72d769df9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# false-discovery-rate-estimation-decoy-search

## Summary

Estimates the false discovery rate of metabolite-spectrum matches (MSMs) in DEREPLICATOR+ database searches by scoring each target fragmentation graph against a matching decoy fragmentation graph, then applying a significance threshold (e.g., 3% FDR) to benchmark rank-1 and top-3 recall and precision.

## When to use

Use when benchmarking DEREPLICATOR+ database search identifications (e.g., on SpectraLibrary or GNPS spectra) and you need a target-decoy FDR estimate to define a significance cutoff for recall/precision measurement.

## When NOT to use

- The target compounds fall outside classes DEREPLICATOR+ can model — the article states 'Similar to all the existing metabolomics tools, DEREPLICATOR+ is not applicable to all the classes of compounds.'
- Spectra are already confidently identified against a small, fully validated reference library where decoy competition would not change the identification set.
- The goal is de novo or spectral-network variant propagation rather than FDR-controlled database search of known metabolites.

## Inputs

- LC-MS/MS spectra in MGF format (mzXML/mzML/mzData convertible to MGF via ProteoWizard msconvert)
- Chemical structure database of known natural products (target metabolites for fragmentation graph construction)
- Known compound annotations (ground truth, e.g., SpectraLibrary compound identities) for computing recall and precision

## Outputs

- Per-spectrum MSM scores (number of unique matched masses) for target and decoy hits
- FDR-estimated set of significant MSMs at the chosen threshold (e.g., 3% FDR)
- Rank-1 and top-3 recall and precision metrics (benchmark targets: 33%/58% rank-1 recall/precision; 48%/85% top-3 recall/precision)

## How to apply

Run DEREPLICATOR+ database search of each spectrum against a reference compound database: for each metabolite M and spectrum S, compute score(M,S) as the number of unique masses in S that annotate a node in M's fragmentation graph. For every target fragmentation graph, DEREPLICATOR+ generates a decoy fragmentation graph so target-decoy competition yields the statistical significance of each MSM and an FDR estimate. Preprocess spectra by retaining only the six top-intensity peaks in each 50 Da window before scoring. Apply the chosen FDR threshold (the benchmark uses 3% FDR, scored by the number of unique masses annotating nodes of the metabolite fragmentation graph) to retain only significant MSMs. Then compute rank-1 recall (correct rank-1 identifications over total spectra), rank-1 precision (correct rank-1 matches over all rank-1 matches), and the analogous top-3 metrics by comparing identifications against known compound annotations. The published validation values to reproduce are rank-1 recall 33% (1802/5473), rank-1 precision 58% (1802/3087), top-3 recall 48% (2614/5473), and top-3 precision 85% (2614/3087) on SpectraLibrary at 3% FDR.

## Related tools

- **DEREPLICATOR+** (Performs the target-decoy database search of mass spectra against metabolite fragmentation graphs and computes MSM statistical significance and FDR) — https://github.com/ablab/npdtools
- **DEREPLICATOR** (Predecessor tool for peptidic natural product (PNP) dereplication included in NPDtools; relevant for PNP-only search pipelines) — https://github.com/ablab/npdtools

## Examples

```
wget https://github.com/ablab/npdtools/releases/download/npdtools-2.5.0/NPDtools-2.5.0-Linux.tar.gz; tar -xzf NPDtools-2.5.0-Linux.tar.gz; python NPDtools-2.5.0-Linux/bin/dereplicator+.py
```

## Evaluation signals

- Rank-1 recall on SpectraLibrary at 3% FDR should reproduce 33% (1802 of 5473 spectra).
- Rank-1 precision on SpectraLibrary at 3% FDR should reproduce 58% (1802 of 3087 identifications).
- Top-3 recall and precision should reproduce 48% (2614/5473) and 85% (2614/3087), respectively.
- Decoy matches must be generated for every target fragmentation graph before FDR is estimated; an FDR value without decoy hits is invalid.
- Spectra must be preprocessed to retain exactly six top-intensity peaks per 50 Da window prior to scoring.

## Limitations

- DEREPLICATOR+ uses simple shared-peaks scoring, which the authors note results in a high false-negative rate; improved scoring was not yet implemented.
- The p-value recursion is only approximate for non-tree (cyclic) fragmentation graphs; unbiased MCMC probability estimation was described as possible but not implemented.
- DEREPLICATOR+ is not applicable to all classes of compounds.
- No version numbers for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS are stated in the article.

## Evidence

- [discussion] decoy generation: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [discussion] scoring definition: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] FDR computation: "Computing The Statistical Significance Of Msms And Evaluating Fdr"
- [discussion] spectral preprocessing: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [readme] tool scope: [evidence span withheld — anchor status: not_found]
