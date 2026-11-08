---
name: breadth-first-fragment-mass-annotation
description: Use when annotating centroided LC-MS/MS spectra against candidate metabolite
  structures via fragmentation graphs, e.g. dereplicating GNPS MGF spectra where each
  matched unique mass must be tallied into an MSM score.
when_to_use_negative:
- The target metabolite class or fragmentation graph is not expressible under the
  C–N/C–O 2-cut and single C–C cut rules (DEREPLICATOR+ is not applicable to all classes
  of compounds)
- The spectra are not centroided or not in an open format (MGF, mzXML, mzML, mzData)
- The goal is identification of peptidic natural products specifically, where the
  original DEREPLICATOR PNP pipeline (rather than fragmentation-graph dereplication)
  applies
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Implements fragmentation graph construction, breadth-first mass annotation,
    and MSM scoring against target and decoy graphs
  repo: github.com/ablab/npdtools
- name: NPDtools
  role: Toolkit packaging the dereplicator+ executable used to run the annotation/scoring
    pipeline on MGF/mzXML/mzML/mzData inputs
  repo: https://github.com/ablab/npdtools
- name: GNPS
  role: Web platform and spectral repository on which DEREPLICATOR+ dereplication
    of molecular networking spectra is run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:789095d9d41bde42b25b37b46aa844986fbf96e9ccf11f30231eafe74f8423e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - GNPS
  - NPDtools
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# breadth-first-fragment-mass-annotation

## Summary

Annotates DEREPLICATOR+ metabolite fragmentation graphs breadth-first with preprocessed tandem MS peaks and scores metabolite-spectrum matches (MSMs) as the number of unique spectrum masses that annotate a graph node, feeding the Poisson-binomial P-value/FDR evaluation.

## When to use

Use when annotating centroided LC-MS/MS spectra against candidate metabolite structures via fragmentation graphs, e.g. dereplicating GNPS MGF spectra where each matched unique mass must be tallied into an MSM score.

## When NOT to use

- The target metabolite class or fragmentation graph is not expressible under the C–N/C–O 2-cut and single C–C cut rules (DEREPLICATOR+ is not applicable to all classes of compounds)
- The spectra are not centroided or not in an open format (MGF, mzXML, mzML, mzData)
- The goal is identification of peptidic natural products specifically, where the original DEREPLICATOR PNP pipeline (rather than fragmentation-graph dereplication) applies

## Inputs

- Centroided LC-MS/MS spectra in MGF, mzXML, mzML, or mzData format (converted to MGF with msconvert if needed)
- Candidate metabolite chemical structure database of known natural products
- Fragmentation graph parameters: depth cutoff k, feasible 2-cut definitions (C–N or C–O)

## Outputs

- MSM Score per metabolite-spectrum pair (count of unique annotating masses)
- Statistical significance per MSM: Poisson-binomial P-Value and FDR estimate
- significant_matches.tsv with SpecFile, Scan, Score, P-Value, FDR, and compound annotation columns

## How to apply

First preprocess the centroided spectrum by retaining only the six top-intensity peaks in each 50 Da window. For each candidate metabolite M, build its fragmentation graph by disconnecting bonds (C–N, C–O, C–C) into connected components labeled by elemental composition, then prune: remove nodes whose depth exceeds k, keep only nodes with a path from the root via at most one 2-cut fragmentation (restricted to feasible C–N or C–O cuts) and at most one C–C bond fragmentation. Breadth-first annotate the fragmentation graph with the preprocessed spectrum masses, and compute score(M,S) as the number of unique masses in spectrum S that annotate a node in the fragmentation graph of M. The resulting per-match Score, together with the Poisson-binomial P-value and FDR estimated over decoy fragmentation graphs, is written to significant_matches.tsv for surviving compound-spectrum matches.

## Related tools

- **DEREPLICATOR+** (Implements fragmentation graph construction, breadth-first mass annotation, and MSM scoring against target and decoy graphs) — github.com/ablab/npdtools
- **NPDtools** (Toolkit packaging the dereplicator+ executable used to run the annotation/scoring pipeline on MGF/mzXML/mzML/mzData inputs) — https://github.com/ablab/npdtools
- **GNPS** (Web platform and spectral repository on which DEREPLICATOR+ dereplication of molecular networking spectra is run)

## Evaluation signals

- The module reproduces the reported depth-annotation counts for validation compounds such as heterocyst glycolipid and chalcomycin
- Every reported compound-spectrum match in significant_matches.tsv carries a Score, a P-Value, and an FDR entry
- Score(M,S) equals the number of unique spectrum masses (after the 6-peaks-per-50-Da preprocessing) that annotate a fragmentation graph node, not a raw peak count
- Pruned fragmentation graphs contain no nodes beyond depth k and no nodes reached via more than one 2-cut (C–N/C–O) or one C–C fragmentation from the root
- Decoy fragmentation graph scores yield a Poisson-binomial P-value distribution consistent with the reported FDR estimation at each P-value level

## Limitations

- Simple shared-peaks scoring results in a high false-negative rate; improved scoring is not yet implemented
- P-value recursion is only approximate for non-tree fragmentation graphs; unbiased estimation via MCMC is described as possible but not implemented
- DEREPLICATOR+ is not applicable to all classes of compounds

## Evidence

- [discussion] Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M.: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] Six top peaks per 50 Da window preprocessing: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] Fragmentation graph node labeling by elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [discussion] Depth pruning rule for fragmentation graphs: "all nodes with depths exceeding k are removed from the graph"
- [readme] NPDtools supported spectra formats for the pipeline: [evidence span withheld — anchor status: external_doc]
