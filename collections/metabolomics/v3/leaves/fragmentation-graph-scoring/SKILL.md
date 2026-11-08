---
name: fragmentation-graph-scoring
description: Use when dereplicating centroided LC-MS/MS spectra (MGF, mzXML, mzML,
  mzData) against a chemical structure database of natural products, especially non-peptidic
  classes (polyketides, terpenes, benzenoids, alkaloids, flavonoids) that peptide-only
  tools like DEREPLICATOR cannot handle.
when_to_use_negative:
- Input spectra are uncentroided profile data or not convertible to an open format
  (MGF, mzXML, mzML, mzData)
- The target compounds fall outside the metabolite classes DEREPLICATOR+ handles —
  the article states it 'is not applicable to all the classes of compounds'
- The analysis only targets peptidic natural products and variants, where DEREPLICATOR
  or VarQuest suffice and fragmentation-graph construction adds unnecessary overhead
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DEREPLICATOR+
  role: Implements fragmentation-graph construction and unique-matched-mass scoring
    of spectra against metabolite structure databases, with decoy-based FDR estimation
  repo: github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Baseline peptide-only (PNP) database search used for comparison at the same
    1% FDR level
  repo: github.com/ablab/npdtools
- name: DEREPLICATOR-G
  role: Baseline variant (C–N bonds all cut rather than amide bonds only) run in parallel
    on the same spectra for comparison
- name: GNPS
  role: Spectral data repository and web platform hosting the DEREPLICATOR+ workflow
    (gnps-theoretical.jsp)
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:74ce04ee36a8c32e7d43abccdd6284c15a17ab32d6b0ea3280abd30ff2374224
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - GNPS
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# fragmentation-graph-scoring

## Summary

Score mass spectra against metabolite fragmentation graphs — nodes formed by disconnecting all metabolite bonds (C–N, C–O, C–C) and labeled by elemental composition — where score(M,S) counts the number of unique masses in S that annotate a fragmentation-graph node. This is the core database-search scoring scheme of DEREPLICATOR+ for dereplicating both peptidic and non-peptidic natural products.

## When to use

Use when dereplicating centroided LC-MS/MS spectra (MGF, mzXML, mzML, mzData) against a chemical structure database of natural products, especially non-peptidic classes (polyketides, terpenes, benzenoids, alkaloids, flavonoids) that peptide-only tools like DEREPLICATOR cannot handle.

## When NOT to use

- Input spectra are uncentroided profile data or not convertible to an open format (MGF, mzXML, mzML, mzData)
- The target compounds fall outside the metabolite classes DEREPLICATOR+ handles — the article states it 'is not applicable to all the classes of compounds'
- The analysis only targets peptidic natural products and variants, where DEREPLICATOR or VarQuest suffice and fragmentation-graph construction adds unnecessary overhead

## Inputs

- Centroided LC-MS/MS spectra in MGF, mzXML, mzML, or mzData format (e.g., GNPS molecular networking datasets)
- Chemical structure database of known natural products (metabolite structures to be converted into fragmentation graphs)

## Outputs

- Scored spectrum–metabolite matches (MSMs) with unique-matched-mass scores
- FDR-estimated identifications (e.g., 410,539 MSMs / 5,336 distinct compounds at 1% FDR on SpectraGNPS)
- Decoy-hit statistics used for FDR estimation

## How to apply

Construct a fragmentation graph for each candidate metabolite by disconnecting all bonds; retain only nodes whose path from the root consists of at most one 2-cut fragmentation limited to feasible C–N or C–O cuts, at most one C–C bond fragmentation, and remove nodes with depth exceeding k. Preprocess each spectrum by retaining only the six top-intensity peaks in each 50 Da window. Annotate fragmentation-graph nodes breadth-first with spectrum peaks and define score(M,S) as the number of unique matched masses annotating graph nodes. Generate a decoy fragmentation graph for each target to compute statistical significance of spectrum–metabolite matches (MSMs) and estimate FDR, filtering results at 1% FDR. Rationale: fragmentation graphs generalize the peptide-bond-cutting model of DEREPLICATOR to diverse metabolites while keeping the search space computationally tractable.

## Related tools

- **DEREPLICATOR+** (Implements fragmentation-graph construction and unique-matched-mass scoring of spectra against metabolite structure databases, with decoy-based FDR estimation) — github.com/ablab/npdtools
- **DEREPLICATOR** (Baseline peptide-only (PNP) database search used for comparison at the same 1% FDR level) — github.com/ablab/npdtools
- **DEREPLICATOR-G** (Baseline variant (C–N bonds all cut rather than amide bonds only) run in parallel on the same spectra for comparison)
- **GNPS** (Spectral data repository and web platform hosting the DEREPLICATOR+ workflow (gnps-theoretical.jsp)) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp

## Examples

```
wget https://github.com/ablab/npdtools/releases/download/npdtools-2.5.0/NPDtools-2.5.0-Linux.tar.gz; tar -xzf NPDtools-2.5.0-Linux.tar.gz; cd NPDtools-2.5.0-Linux; bin/dereplicator+.py --spectra spectra.mgf --database structures.txt --fdr 0.01
```

## Evaluation signals

- Reported identifications must reproduce the published scale at 1% FDR: 410,539 MSMs and 5,336 distinct compounds over the 248.1-million-spectrum SpectraGNPS dataset (score threshold of 12)
- Identification yield should exceed the DEREPLICATOR (41,403 MSMs / 974 compounds) and DEREPLICATOR-G (82,260 MSMs / 1,417 compounds) baselines by roughly an order of magnitude
- Every spectrum match should be validated against decoy fragmentation graphs with the overall FDR held at or below 1%
- Preprocessed spectra should contain exactly six top-intensity peaks per 50 Da window
- Fragmentation graphs should contain only nodes within depth k whose root path has at most one feasible 2-cut (C–N or C–O) and at most one C–C fragmentation

## Limitations

- The simple shared-peaks scoring scheme results in a high false-negative rate; an improved scoring is not implemented
- The p-value recursion is only approximate for non-tree fragmentation graphs; unbiased MCMC estimation is described as possible but not implemented
- DEREPLICATOR+ is not applicable to all classes of compounds
- The vast majority of GNPS spectra remain uninterpreted (identified spectra are only 0.2% of SpectraGNPS) — the 'dark matter of metabolomics'

## Evidence

- [results] scoring defined as the number of unique matched masses annotating fragmentation-graph nodes: [evidence span withheld — anchor status: not_found]
- [discussion] 50 Da peak filtering: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] score definition: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] feasible 2-cut restriction: "2-cut fragmentations are limited to feasible 2-cuts"
- [discussion] high false-negative limitation: [evidence span withheld — anchor status: not_found]
- [readme] supported input formats: [evidence span withheld — anchor status: not_found]
