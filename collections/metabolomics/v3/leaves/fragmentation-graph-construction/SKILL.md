---
name: fragmentation-graph-construction
description: Use when dereplicating centroided LC-MS/MS spectra against a database
  of known metabolite structures and you need to model in-silico fragment ions of
  candidate metabolites beyond peptidic natural products (e.g., polyketides, terpenes,
  alkaloids).
when_to_use_negative:
- The metabolites of interest are exclusively peptidic natural products — original
  DEREPLICATOR (PNP-only) is the appropriate simpler tool
- You need high-accuracy identification relying on rich scoring (e.g., peak-intensity
  or mass-accuracy aware matching) — DEREPLICATOR+ uses simple shared-peaks scoring,
  which the authors note causes a high false-negative rate
- The spectra are profile mode or not centroided — the pipeline requires centroided
  input
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Implements the fragmentation graph construction, spectral annotation, and
    decoy-graph-based statistical scoring
  repo: github.com/ablab/npdtools
- name: GNPS
  role: Platform hosting the DEREPLICATOR+ web workflow and the molecular networking
    data searched with the constructed graphs
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: NPDtools
  role: Toolkit package distributing the dereplicator+.py executable and supporting
    scripts (db_preprocessing.py, print_score, print_structure)
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:11af097dfb01477efd09ed34c7708a543600a98e48a8f34fac82334b645fd895
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - GNPS
  - NPDtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fragmentation-graph-construction

## Summary

Constructs DEREPLICATOR+ fragmentation graphs by disconnecting all metabolite bonds into connected components labeled by elemental composition, then pruning the graph so that only nodes reachable from the root via at most one feasible 2-cut (C–N or C–O) and at most one C–C fragmentation remain. These graphs are the substrate for scoring metabolite-spectrum matches (MSMs) by counting uniquely annotated spectrum masses.

## When to use

Use when dereplicating centroided LC-MS/MS spectra against a database of known metabolite structures and you need to model in-silico fragment ions of candidate metabolites beyond peptidic natural products (e.g., polyketides, terpenes, alkaloids).

## When NOT to use

- The metabolites of interest are exclusively peptidic natural products — original DEREPLICATOR (PNP-only) is the appropriate simpler tool
- You need high-accuracy identification relying on rich scoring (e.g., peak-intensity or mass-accuracy aware matching) — DEREPLICATOR+ uses simple shared-peaks scoring, which the authors note causes a high false-negative rate
- The spectra are profile mode or not centroided — the pipeline requires centroided input

## Inputs

- Centroided LC-MS/MS spectra in MGF (or mzXML/mzML/mzData, converted to MGF via ProteoWizard msconvert)
- Chemical structure database of known natural products (e.g., Dictionary of Natural Products, AntiMarin, KEGG entries)
- Depth cutoff parameter k for node pruning

## Outputs

- Target fragmentation graph per candidate metabolite (nodes labeled by elemental composition)
- Decoy fragmentation graph per target graph
- MSM scores: number of unique spectrum masses annotating fragmentation graph nodes, with P-Value and FDR entries in significant_matches.tsv

## How to apply

For each candidate metabolite, disconnect all bonds (C–N, C–O, C–C) so the connected components become nodes of the metabolite graph labeled by elemental composition. Prune by removing all nodes with depth exceeding k, and retain only nodes whose path from the root consists of at most one 2-cut fragmentation (restricted to feasible C–N or C–O cuts) and at most one C–C bond fragmentation. Preprocess spectra first by retaining only the six top-intensity peaks in each 50 Da window, then breadth-first annotate the fragmentation graph with the preprocessed spectrum masses; define score(M,S) as the number of unique masses in S annotating a node of M's fragmentation graph. Also generate a decoy fragmentation graph for each target graph to support Poisson-binomial p-value and FDR estimation. The pruning restrictions keep the graph computationally tractable while capturing dominant single- and double-bond fragmentation events.

## Related tools

- **DEREPLICATOR+** (Implements the fragmentation graph construction, spectral annotation, and decoy-graph-based statistical scoring) — github.com/ablab/npdtools
- **GNPS** (Platform hosting the DEREPLICATOR+ web workflow and the molecular networking data searched with the constructed graphs) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **NPDtools** (Toolkit package distributing the dereplicator+.py executable and supporting scripts (db_preprocessing.py, print_score, print_structure)) — https://github.com/ablab/npdtools

## Evaluation signals

- Reproduce the reported depth-annotation counts for validation metabolites such as heterocyst glycolipid and chalcomycin
- Every surviving compound-spectrum match in significant_matches.tsv carries a Score, P-Value, and FDR entry
- No node in a built fragmentation graph has a path from the root containing more than one 2-cut fragmentation or more than one C–C bond fragmentation
- Spectrum preprocessing retains exactly six top-intensity peaks per 50 Da window before annotation
- score(M,S) equals the count of unique spectrum masses annotating at least one node of M's fragmentation graph (no duplicate masses counted)

## Limitations

- The simple shared-peaks scoring scheme results in a high false-negative rate; improved scoring is not yet implemented
- The p-value recursion is only approximate for non-tree fragmentation graphs; unbiased MCMC probability estimation is described as possible but not implemented
- DEREPLICATOR+ is not applicable to all classes of compounds
- No version numbers are stated for DEREPLICATOR+ in the article, complicating reproducibility

## Evidence

- [discussion] Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [discussion] To address this complication, DEREPLICATOR+ introduces the concept of the fragmentation graph.: "To address this complication, DEREPLICATOR+ introduces the concept of the fragmentation graph."
- [discussion] all nodes with depths exceeding k are removed from the graph: "all nodes with depths exceeding k are removed from the graph"
- [discussion] only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph: "only nodes with a path from the root consisting of maximum one 2-cut fragmentation are retained in the graph"
- [discussion] Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M.: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [readme] Dereplicator+ — a tool for identification of metabolites (both peptidic and non-peptidic) through database search of mass spectra: [evidence span withheld — anchor status: not_found]
