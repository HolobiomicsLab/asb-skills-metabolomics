---
name: metabolite-spectrum-match-scoring
description: Use when tandem MS spectra (MGF/mzXML/mzML/mzData) must be searched against
  a natural product structure database and matches filtered at a target-decoy FDR
  level, or when reproducing DEREPLICATOR+ compound/MSM counts.
when_to_use_negative:
- Input is MS1-only data or an already-computed feature/abundance table rather than
  tandem MS spectra — no fragmentation peaks exist to score against fragmentation
  graphs.
- The target molecules are better served by genome-based metabologenomic pipelines
  (e.g. RiPP discovery via MetaMiner) rather than spectrum-to-structure database search.
- The compound classes of interest are known to be outside DEREPLICATOR+'s coverage
  — the article states it 'is not applicable to all the classes of compounds.'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: 'core search engine: builds fragmentation and decoy fragmentation graphs,
    computes MSM scores, P-values, and FDR; available stand-alone or as a GNPS web
    application'
  repo: http://mohimanilab.cbd.cmu.edu/software/
- name: DEREPLICATOR
  role: peptidic-natural-product-only predecessor used as the comparison baseline
    for dereplication counts
  repo: https://github.com/ablab/npdtools
- name: GNPS
  role: web platform hosting the DEREPLICATOR+ workflow and the mass spectrometry
    repository searched for dereplication
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: msconvert (ProteoWizard)
  role: converts raw/vendor spectra formats to centroided MGF, the format consumed
    by DEREPLICATOR+
  repo: http://proteowizard.sourceforge.net/tools/msconvert.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:f35ff438ce7f7aa22f32cfde32ea7d007965e67718f2fd83356f00d8c185ad21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - GNPS
  - msconvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-spectrum-match-scoring

## Summary

Scores centroided LC-MS/MS spectra against fragmentation graphs of known natural product structures using DEREPLICATOR+, computes MSM P-values, and estimates target-decoy FDR to dereplicate metabolites at user-chosen FDR thresholds (e.g. 0%, 1%, 3%).

## When to use

Use when tandem MS spectra (MGF/mzXML/mzML/mzData) must be searched against a natural product structure database and matches filtered at a target-decoy FDR level, or when reproducing DEREPLICATOR+ compound/MSM counts.

## When NOT to use

- Input is MS1-only data or an already-computed feature/abundance table rather than tandem MS spectra — no fragmentation peaks exist to score against fragmentation graphs.
- The target molecules are better served by genome-based metabologenomic pipelines (e.g. RiPP discovery via MetaMiner) rather than spectrum-to-structure database search.
- The compound classes of interest are known to be outside DEREPLICATOR+'s coverage — the article states it 'is not applicable to all the classes of compounds.'

## Inputs

- centroided LC-MS/MS spectra in MGF, mzXML, mzML, or mzData format (e.g. MassIVE MSV000078604, MSV000078839)
- chemical structure database of known natural products
- target FDR thresholds (e.g. 0%, 1%, 3%)

## Outputs

- list of metabolite-spectrum matches (MSMs) with scores, P-values, and FDR annotations (e.g. dereplicator_plus_spectraactiseq_results.tsv)
- counts of distinct identified compounds and total MSMs at each FDR threshold

## How to apply

Convert raw spectra to centroided MGF with the msconvert utility from ProteoWizard, then preprocess each spectrum by retaining only the six top-intensity peaks in each 50 Da window. For each candidate metabolite, DEREPLICATOR+ constructs a fragmentation graph whose nodes are connected components labeled by elemental composition, with nodes of depth exceeding k removed and paths limited to at most one 2-cut fragmentation (restricted to feasible C–N or C–O cuts) and one C–C bond fragmentation. Generate a decoy fragmentation graph for each target fragmentation graph to support target-decoy FDR estimation, then score each metabolite–spectrum pair as score(M,S), the number of unique masses in the spectrum that annotate a node in the metabolite's fragmentation graph. Compute statistical significance (P-values) for MSMs, evaluate FDR, and select matches at the 0%, 1%, and 3% FDR levels before counting distinct compounds and total MSMs per threshold. Optionally enlarge the MSM set via molecular networking by selecting connected components with at least one identified spectrum to annotate variants.

## Related tools

- **DEREPLICATOR+** (core search engine: builds fragmentation and decoy fragmentation graphs, computes MSM scores, P-values, and FDR; available stand-alone or as a GNPS web application) — http://mohimanilab.cbd.cmu.edu/software/
- **DEREPLICATOR** (peptidic-natural-product-only predecessor used as the comparison baseline for dereplication counts) — https://github.com/ablab/npdtools
- **GNPS** (web platform hosting the DEREPLICATOR+ workflow and the mass spectrometry repository searched for dereplication) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **msconvert (ProteoWizard)** (converts raw/vendor spectra formats to centroided MGF, the format consumed by DEREPLICATOR+) — http://proteowizard.sourceforge.net/tools/msconvert.html

## Examples

```
msconvert data.raw --mgf && <npdtools_installation_dir>/bin/dereplicator+.py --spectra spectra.mgf --database natural_products.db
```

## Evaluation signals

- Reproduced counts match reported values: 488 compounds (8194 MSMs) at 1% FDR, 154 compounds (2666 MSMs) at 0% FDR, and 1024 compounds at 3% FDR on SpectraActiSeq.
- Target-decoy balance: the decoy fragmentation graphs yield decoy MSM counts consistent with the stated FDR level at each threshold.
- Every reported MSM's score equals the number of unique spectrum masses annotating a node in the metabolite's fragmentation graph.
- Output TSV contains one row per MSM with P-value and FDR-level annotation at 0%, 1%, and 3%.
- Spectra are preprocessed to exactly six top-intensity peaks per 50 Da window before scoring.

## Limitations

- The simple shared-peaks scoring scheme results in a high false-negative rate; an improved scoring function is not yet implemented.
- P-value recursion is only approximate for non-tree fragmentation graphs; unbiased estimation via MCMC is described as possible but not implemented.
- DEREPLICATOR+ is not applicable to all classes of compounds, similar to existing metabolomics tools.
- No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS, complicating exact reproduction.

## Evidence

- [discussion] DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da.: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] score definition as unique matched masses: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] decoy generation for FDR estimation: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [methods] msconvert conversion to MGF: "uses msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF"
- [abstract] order-of-magnitude improvement over prior dereplication: "identify an order of magnitude more natural products than previous dereplication efforts"
