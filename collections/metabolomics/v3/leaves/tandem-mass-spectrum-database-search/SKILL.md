---
name: tandem-mass-spectrum-database-search
description: Use when LC-MS/MS spectra (MGF, mzXML, mzML, or mzData) need to be identified
  against a database of known natural product structures, especially when exact-formula
  dereplication by precursor mass alone is ambiguous.
when_to_use_negative:
- The compounds of interest fall outside the metabolite classes DEREPLICATOR+ covers
  — the paper states 'DEREPLICATOR+ is not applicable to all the classes of compounds.'
- The goal is de novo structure elucidation rather than dereplication against known
  structures.
- Spectra are peak-picked poorly or profile-mode, since NPDtools requires centroided
  spectra in an open format.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DEREPLICATOR+
  role: Primary database search tool matching spectra against metabolite fragmentation
    graphs with decoy-based FDR estimation
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Predecessor tool limited to peptidic natural products (PNPs); used for comparison
    when the target class is PNPs only
  repo: https://github.com/ablab/npdtools
- name: NPDtools
  role: Toolkit packaging dereplicator+.py, dereplicator.py, and supporting scripts
    (db_preprocessing.py, print_score, print_structure) for running the search
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:438f4790f0df01b44b90e102b0e047f9459d457a4cd45427f0ea51b7a109cd78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - NPDtools
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# tandem-mass-spectrum-database-search

## Summary

Dereplicate LC-MS/MS spectra by searching them against chemical structure databases using DEREPLICATOR+, which matches spectra to metabolite fragmentation graphs and estimates FDR via decoy matches. Extends peptidic natural product search to polyketides, terpenes, benzenoids, alkaloids, flavonoids, and other classes.

## When to use

Use when LC-MS/MS spectra (MGF, mzXML, mzML, or mzData) need to be identified against a database of known natural product structures, especially when exact-formula dereplication by precursor mass alone is ambiguous.

## When NOT to use

- The compounds of interest fall outside the metabolite classes DEREPLICATOR+ covers — the paper states 'DEREPLICATOR+ is not applicable to all the classes of compounds.'
- The goal is de novo structure elucidation rather than dereplication against known structures.
- Spectra are peak-picked poorly or profile-mode, since NPDtools requires centroided spectra in an open format.

## Inputs

- Centroided LC-MS/MS spectra in MGF, mzXML, mzML, or mzData format
- Chemical structure database of known natural products (e.g., DNP, AntiMarin, KEGG, PubChem-derived)

## Outputs

- Ranked metabolite-spectrum matches (MSMs) with FDR-estimated significance scores
- Identifications of natural products and their variants per input spectrum

## How to apply

Convert spectra to MGF if needed (e.g., with msconvert from ProteoWizard) and run the Dereplicator+ pipeline from NPDtools (dereplicator+.py) against a chemical structure database of known natural products. DEREPLICATOR+ constructs a fragmentation graph for each metabolite by disconnecting bonds (C–N, C–O, C–C), removes nodes with depth exceeding k and restricts 2-cut fragmentations to feasible C–N/C–O cuts and at most one C–C fragmentation per path; spectra are preprocessed by retaining only the six top intensity peaks in each 50 Da window. Each spectrum is scored against target fragmentation graphs by the number of unique masses annotating graph nodes, and a decoy fragmentation graph is generated per target to compute MSM statistical significance and FDR. Retain metabolite-spectrum matches at the chosen FDR threshold (e.g., 3% as used on the SpectraLibrary benchmark) and report rank-1 recall/precision (33% recall, 58% precision at 3% FDR on SpectraLibrary) or top-3 metrics (48% recall, 85% precision) against known compound annotations.

## Related tools

- **DEREPLICATOR+** (Primary database search tool matching spectra against metabolite fragmentation graphs with decoy-based FDR estimation) — https://github.com/ablab/npdtools
- **DEREPLICATOR** (Predecessor tool limited to peptidic natural products (PNPs); used for comparison when the target class is PNPs only) — https://github.com/ablab/npdtools
- **NPDtools** (Toolkit packaging dereplicator+.py, dereplicator.py, and supporting scripts (db_preprocessing.py, print_score, print_structure) for running the search) — https://github.com/ablab/npdtools

## Examples

```
wget https://github.com/ablab/npdtools/releases/download/npdtools-2.5.0/NPDtools-2.5.0-Linux.tar.gz && tar -xzf NPDtools-2.5.0-Linux.tar.gz && ./bin/dereplicator+.py --spectra spectra.mgf --database dnp structures
```

## Evaluation signals

- At 3% FDR on the SpectraLibrary benchmark, rank-1 recall should be ~33% (1802/5473) and rank-1 precision ~58% (1802/3087) when success is a rank-1 correct prediction.
- Top-3 recall should reach ~48% (2614/5473) and top-3 precision ~85% (2614/3087) on the same benchmark.
- Estimated FDR from target-decoy matches should be near the applied threshold (e.g., ~3%) — verify decoy MSM counts versus retained target MSMs.
- Output score for each MSM equals the number of unique spectrum masses annotating nodes of the matched metabolite's fragmentation graph; spot-check scores against manual peak-matching.
- All returned structures should be verifiable against the queried chemical database via print_structure output; no identifications should reference compounds absent from the database.

## Limitations

- DEREPLICATOR+ uses simple shared-peaks scoring, which results in a high false-negative rate; improved scoring is described as not yet implemented.
- P-value computation is only approximate for non-tree fragmentation graphs; unbiased MCMC probability estimation is possible but not implemented.
- DEREPLICATOR+ is not applicable to all classes of compounds.
- No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS; results may vary with tool version.
- The vast majority of GNPS spectra remain uninterpreted (the 'dark matter of metabolomics'), so many real-world spectra will not match any database entry.

## Evidence

- [discussion] scoring by unique matched masses: "we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] spectrum preprocessing filter: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] decoy-based FDR estimation: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [readme] supported input formats: [evidence span withheld — anchor status: not_found]
- [discussion] fragmentation graph construction: "Disconnecting all metabolite bonds in a metabolite breaks it into connected component that form the nodes of the metabolite graph labeled by their elemental composition"
- [abstract] class coverage extension: "extends them for identification of polyketides, terpenes, benzenoids, alkaloids, flavonoids, and other classes of natural products"
