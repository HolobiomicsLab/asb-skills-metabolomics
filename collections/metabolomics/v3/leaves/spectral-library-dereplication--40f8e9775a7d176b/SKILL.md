---
name: spectral-library-dereplication
description: Use when annotated MS/MS spectra must be matched against a database of
  known metabolite structures to benchmark dereplication tools, or when tuning precursor
  mass tolerance (e.g., 0.5 Da vs 0.02 Da) to maximize top-rank identification rates.
when_to_use_negative:
- The target metabolites are not represented in the reference structure database (e.g.,
  DEREPLICATOR alone is limited to peptidic natural products and cannot identify polyketides,
  terpenes, and other non-peptidic classes)
- The input spectra are profile-mode or in a closed vendor format without conversion
  (NPDtools requires centroided spectra in MGF, mzXML, mzML, or mzData)
- The goal is de novo characterization of unknown 'dark matter' spectra rather than
  dereplication against known structures
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DEREPLICATOR+
  role: Primary database search tool that dereplicates spectra against diverse metabolite
    classes (polyketides, terpenes, benzenoids, alkaloids, flavonoids) via fragmentation-graph
    scoring
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR
  role: Baseline search tool restricted to peptidic natural products, used as a comparison
    point at both mass tolerances
  repo: https://github.com/ablab/npdtools
- name: GNPS
  role: Source of spectral library structures merged with DNP to build the 86,665-structure
    reference database; also hosts DEREPLICATOR+ as a web application
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: msconvert (ProteoWizard)
  role: Converts spectra in other formats to MGF before NPDtools search
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:de6efd9794f6f49fec3e314e44107090ad6b2b5df964a6d83de5326828df509d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - GNPS
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# spectral-library-dereplication

## Summary

Dereplicate LC-MS/MS spectra of known natural products by searching them as mass spectra against large chemical structure databases (e.g., a merged DNP + GNPS library of 86,665 structures) using the DEREPLICATOR family of tools, and quantify top-rank identification accuracy at different precursor mass tolerances.

## When to use

Use when annotated MS/MS spectra must be matched against a database of known metabolite structures to benchmark dereplication tools, or when tuning precursor mass tolerance (e.g., 0.5 Da vs 0.02 Da) to maximize top-rank identification rates.

## When NOT to use

- The target metabolites are not represented in the reference structure database (e.g., DEREPLICATOR alone is limited to peptidic natural products and cannot identify polyketides, terpenes, and other non-peptidic classes)
- The input spectra are profile-mode or in a closed vendor format without conversion (NPDtools requires centroided spectra in MGF, mzXML, mzML, or mzData)
- The goal is de novo characterization of unknown 'dark matter' spectra rather than dereplication against known structures

## Inputs

- Annotated benchmark LC-MS/MS spectra (5473 SpectraLibrary spectra) in MGF, mzXML, mzML, or mzData format (centroided)
- Chemical structure database: Dictionary of Natural Products (DNP) merged with GNPS spectral library structures (86,665 structures)
- Precursor mass tolerance settings (0.5 Da and 0.02 Da)
- Known metabolite annotations per spectrum as ground-truth labels

## Outputs

- Top-ranked metabolite-spectrum matches per spectrum, tool, and tolerance
- top_rank_percentages.csv tabulating top-rank percentages and absolute hit counts (e.g., DEREPLICATOR+ 34%/1878 at 0.5 Da and 44%/2414 at 0.02 Da)
- Per-tool accuracy comparison across precursor mass tolerances

## How to apply

Compile the benchmark set of annotated spectra (e.g., 5473 SpectraLibrary spectra) whose known metabolites serve as ground truth, and merge reference structures — here the Dictionary of Natural Products (~300 thousand compounds) with GNPS spectral library structures — into a combined database. Ensure spectra are centroided MGF (convert other formats with msconvert from ProteoWizard), since NPDtools parses MGF natively. Run each tool (DEREPLICATOR+, DEREPLICATOR-G, DEREPLICATOR) via NPDtools at each precursor mass tolerance (0.5 Da and 0.02 Da); DEREPLICATOR+ preprocesses spectra by retaining only the six top intensity peaks per 50 Da window and scores candidates by unique masses annotating a fragmentation graph, while decoy fragmentation graphs support statistical significance/FDR estimation. Rank candidate metabolite-spectrum matches by score per tool and tolerance, and count how many spectra have the correct annotated metabolite as the top-ranked hit. Rationale: tightening precursor tolerance from 0.5 Da to 0.02 Da substantially reduces false top-rank matches (e.g., DEREPLICATOR+ rises from 34%/1878 to 44%/2414 correct top-rank IDs), so both tolerances should be reported to expose the tolerance sensitivity of each tool.

## Related tools

- **DEREPLICATOR+** (Primary database search tool that dereplicates spectra against diverse metabolite classes (polyketides, terpenes, benzenoids, alkaloids, flavonoids) via fragmentation-graph scoring) — https://github.com/ablab/npdtools
- **DEREPLICATOR** (Baseline search tool restricted to peptidic natural products, used as a comparison point at both mass tolerances) — https://github.com/ablab/npdtools
- **GNPS** (Source of spectral library structures merged with DNP to build the 86,665-structure reference database; also hosts DEREPLICATOR+ as a web application) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **msconvert (ProteoWizard)** (Converts spectra in other formats to MGF before NPDtools search)

## Examples

```
wget https://github.com/ablab/npdtools/releases/download/npdtools-2.5.0/NPDtools-2.5.0-Linux.tar.gz; tar -xzf NPDtools-2.5.0-Linux.tar.gz; <npdtools_installation_dir>/bin/dereplicator+.py --spectra SpectraLibrary_5473.mgf --database DNP_GNPS_86665 --precursor-mass-tolerance 0.02
```

## Evaluation signals

- Top-rank percentages and absolute counts must match the benchmark: DEREPLICATOR+ 34% (1878/5473) at 0.5 Da and 44% (2414) at 0.02 Da; DEREPLICATOR-G 30% (1647) and 40% (2216); DEREPLICATOR 17% (964) and 25% (1362)
- For every tool, the top-rank rate at 0.02 Da tolerance must exceed the rate at 0.5 Da tolerance (tightening tolerance improves accuracy)
- The combined reference library must contain 86,665 structures (DNP + GNPS structures)
- All 5473 SpectraLibrary spectra are searched per tool per tolerance; every spectrum yields a ranked candidate list with the known annotation checked at rank 1
- Statistical significance of metabolite-spectrum matches is evaluated against decoy fragmentation graphs with FDR estimation

## Limitations

- DEREPLICATOR+ uses simple shared peaks scoring, which results in a high false-negative rate
- DEREPLICATOR+ is not applicable to all classes of compounds
- P-value recursion is only approximate for non-tree fragmentation graphs; unbiased MCMC estimation is described as possible but not implemented
- No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS in the article

## Evidence

- [readme] Database search pipelines (Dereplicator, VarQuest and Dereplicator+) also require a chemical structure database of known natural products.: [evidence span withheld — anchor status: external_doc]
- [readme] Spectra files must be centroided and be in an open spectrum format: [evidence span withheld — anchor status: external_doc]
- [discussion] DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da.: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [methods] uses msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF: "uses msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF"
- [discussion] DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph.: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [intro] While DEREPLICATOR search of GNPS identified hundreds of peptides and their variants, it is limited to dereplicating PNPs: "While DEREPLICATOR search of GNPS identified hundreds of peptides and their variants, it is limited to dereplicating PNPs and cannot identify other classes of natural products such as polyketides and"
