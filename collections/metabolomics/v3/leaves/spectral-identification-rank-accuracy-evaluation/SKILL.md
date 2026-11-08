---
name: spectral-identification-rank-accuracy-evaluation
description: Use when a tandem mass spectrum database-search tool (e.g. CycloBranch)
  returns ranked structure identifications and you must quantify how often the known
  compound appears at rank 1 or within the top 10.
when_to_use_negative:
- The task is de novo spectrum annotation/sequencing rather than database-search dereplication
  against a known sequence database.
- The analytes are non-peptidic natural products or arbitrary small molecules outside
  the NRP/PK/siderophore sequence space supported by the MassSpecBlocks-exported database.
- Input is not tandem mass spectral data (e.g. MS1-only feature tables or chromatographic
  profiles), so no b/a-ion fragment matching or ranked identification report can be
  produced.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassSpecBlocks
  role: Builds the custom NRP/PK sequence and building-block database and exports
    it into the plain text format consumed by CycloBranch for the dereplication search
  repo: github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Runs the database search (dereplication) of experimental tandem mass spectra
    against the imported sequence database and produces the ranked identification
    report used to compute top-1/top-10 accuracy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:9876f9ba994b326a4c34aea96441d7dbd5b1081d12e13792e144535f2cd19020
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spectral-identification-rank-accuracy-evaluation

## Summary

Benchmarking workflow that runs a tandem-MS dereplication (database) search with CycloBranch against a custom NRP/PK sequence database and computes top-1 and top-10 accuracy of correct structure retrieval from the ranked identification report.

## When to use

Use when a tandem mass spectrum database-search tool (e.g. CycloBranch) returns ranked structure identifications and you must quantify how often the known compound appears at rank 1 or within the top 10.

## When NOT to use

- The task is de novo spectrum annotation/sequencing rather than database-search dereplication against a known sequence database.
- The analytes are non-peptidic natural products or arbitrary small molecules outside the NRP/PK/siderophore sequence space supported by the MassSpecBlocks-exported database.
- Input is not tandem mass spectral data (e.g. MS1-only feature tables or chromatographic profiles), so no b/a-ion fragment matching or ranked identification report can be produced.

## Inputs

- Experimental high-resolution tandem mass spectra (deposited spectra of compounds with known identities)
- NRP/PK sequence and building-block database exported from MassSpecBlocks into CycloBranch plain text format (e.g. 'Nonribosomal Peptides and Siderophores' container: 85 building blocks, 146 sequences)
- Known compound identity (ground truth) per deposited spectrum

## Outputs

- CycloBranch output report (JSON/plain text) listing ranked structure identifications per spectrum
- Top-1 accuracy of correct structure retrieval
- Top-10 accuracy of correct structure retrieval

## How to apply

First build the reference database in MassSpecBlocks (default container 'Nonribosomal Peptides and Siderophores': 85 building blocks and 146 sequences) and export it into the plain text format accepted by CycloBranch. Run the CycloBranch dereplication search on the deposited high-resolution tandem mass spectra with precursor and fragment ion m/z error tolerances of 5 ppm, a minimum threshold of relative peak intensity of zero, and restriction to b-ions and a-ions with at most one neutral loss of H2O or NH3 (to unify configuration with NRPro); enable the precursor mass filter. Parse the CycloBranch output report (JSON/plain text) that lists ranked structure identifications per spectrum. Compare each ranked identification list against the known compound identity of its spectrum to compute top-1 and top-10 accuracy. The rationale is that desferrioxamine B was retrieved as a single hit against the 146-sequence database when the precursor mass filter was enabled, so rank-based accuracy reflects true dereplication performance rather than candidate-list noise.

## Related tools

- **MassSpecBlocks** (Builds the custom NRP/PK sequence and building-block database and exports it into the plain text format consumed by CycloBranch for the dereplication search) — github.com/privrja/MassSpecBlocks
- **CycloBranch** (Runs the database search (dereplication) of experimental tandem mass spectra against the imported sequence database and produces the ranked identification report used to compute top-1/top-10 accuracy)

## Evaluation signals

- With the desferrioxamine B spectrum searched against the 146-sequence NRP/siderophore database and the precursor mass filter enabled, CycloBranch reports desferrioxamine B as a single hit.
- Computed top-1 and top-10 accuracies match the benchmark values reported in the paper's results section for the 5 ppm CycloBranch configuration.
- The output report corresponds to the report format shown in Additional file 1: Fig. S13.
- Search configuration check: precursor and fragment ion m/z error tolerances are 5 ppm, minimum relative peak intensity threshold is zero, and only b-ions and a-ions with at most one neutral loss of H2O or NH3 were searched.
- The exported database contains 85 building blocks and 146 sequences before the search is run.

## Limitations

- Results depend on the precursor mass filter state: a precursor mass filter was enabled for one desferrioxamine B report and disabled for another (Fig. S13 shows the enabled case), so rank accuracy must be reported for a clearly stated filter configuration.
- Restricting the search to b-ions and a-ions with only one neutral loss of H2O or NH3 (chosen to unify configuration with NRPro) ignores other ion series and additional neutral losses, which can lower retrieval performance.
- The accuracy metric is only meaningful relative to the supplied custom database; NRPro users cannot create custom databases of building blocks and NRP sequences, so the comparison is anchored to the CycloBranch/MassSpecBlocks setup.

## Evidence

- [results] Configurable search parameters used for the benchmark: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Ion-type restriction rationale: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [abstract] Database export step feeding the search: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [results] Report-format validation reference: "In Additional file 1: Fig. S13, we show an output report if a precursor mass filter was enabled."
- [readme] Tool role from repository documentation: "The application provides an export format for open source program [CycloBranch](https://ms.biomed.cas.cz/cyclobranch/docs/html/) from Jiří Novák"
