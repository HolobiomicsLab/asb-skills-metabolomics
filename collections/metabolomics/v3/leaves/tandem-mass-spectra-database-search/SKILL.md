---
name: tandem-mass-spectra-database-search
description: Use when you have experimental high-resolution tandem MS/MS spectra of
  putative NRP/PK natural products and a curated sequence database (e.g., 146 NRP/siderophore
  sequences, 85 building blocks) and need ranked compound identifications or dereplication
  accuracy metrics.
when_to_use_negative:
- When the query compounds are non-peptidic natural products better handled by Dereplicator+
  in-silico fragmentation rather than building-block sequences
- When users need custom NRP/PK building-block/sequence databases but NRPro is the
  target engine — NRPro users cannot create custom databases of building blocks and
  NRP sequences
- When spectra are low-resolution, since the method is validated with 5 ppm precursor
  and fragment ion m/z error tolerances
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MassSpecBlocks
  role: Constructs the custom NRP/PK building-block and sequence database (SMILES
    -> building-block sequences) and exports it into the plain text format consumed
    by CycloBranch
  repo: github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Performs the database-search dereplication of the tandem mass spectra against
    the imported sequence database and produces the ranked identification report
- name: NRPro
  role: Comparison tool whose configuration (5 ppm tolerances, b/a-ions, one H2O or
    NH3 neutral loss) the CycloBranch search settings are unified with
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:8a34bdc5d58c301ea63f2bd679dc7e19b7cf73d16c7580faddebeb6c3b652819
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tandem-mass-spectra-database-search

## Summary

Dereplication of high-resolution tandem mass spectra of NRPs/polyketides by searching them against a custom building-block/sequence database in CycloBranch, ranking candidate structures and computing top-1 and top-10 retrieval accuracy.

## When to use

Use when you have experimental high-resolution tandem MS/MS spectra of putative NRP/PK natural products and a curated sequence database (e.g., 146 NRP/siderophore sequences, 85 building blocks) and need ranked compound identifications or dereplication accuracy metrics.

## When NOT to use

- When the query compounds are non-peptidic natural products better handled by Dereplicator+ in-silico fragmentation rather than building-block sequences
- When users need custom NRP/PK building-block/sequence databases but NRPro is the target engine — NRPro users cannot create custom databases of building blocks and NRP sequences
- When spectra are low-resolution, since the method is validated with 5 ppm precursor and fragment ion m/z error tolerances

## Inputs

- Experimental high-resolution tandem mass spectra of known natural products (e.g., desferrioxamine B)
- Custom NRP/PK sequence and building-block database exported from MassSpecBlocks in CycloBranch plain text format
- Known compound identities for the deposited spectra (ground-truth labels for accuracy computation)

## Outputs

- CycloBranch output report (JSON/plain text) listing ranked structure identifications per spectrum
- Top-1 and top-10 accuracy of correct structure retrieval against the benchmark values

## How to apply

Build the custom NRP/PK sequence and building-block database in MassSpecBlocks (e.g., the default 'Nonribosomal Peptides and Siderophores' container with 85 building blocks and 146 sequences) and export it into the plain text format accepted by CycloBranch. In CycloBranch, configure the precursor and fragment ion m/z error tolerances to 5 ppm, set the minimum threshold of relative peak intensity to zero, and restrict the search to b-ions and a-ions allowing at most one neutral loss of H2O or NH3 (to unify the configuration with NRPro); enable the precursor mass filter when precursor filtering is desired. Run the search of the deposited tandem mass spectra against the imported database and parse the output report (JSON/plain text ranked identifications per spectrum). Compute top-1 and top-10 accuracy by comparing the ranked identifications with the known compound identities of the deposited spectra.

## Related tools

- **MassSpecBlocks** (Constructs the custom NRP/PK building-block and sequence database (SMILES -> building-block sequences) and exports it into the plain text format consumed by CycloBranch) — github.com/privrja/MassSpecBlocks
- **CycloBranch** (Performs the database-search dereplication of the tandem mass spectra against the imported sequence database and produces the ranked identification report)
- **NRPro** (Comparison tool whose configuration (5 ppm tolerances, b/a-ions, one H2O or NH3 neutral loss) the CycloBranch search settings are unified with)

## Evaluation signals

- Top-1 and top-10 accuracy of correct structure retrieval must match the benchmark values reported in the paper's results section for the 5 ppm CycloBranch configuration
- With the desferrioxamine B spectrum searched against the 146-sequence database with the precursor mass filter enabled, CycloBranch must report desferrioxamine B as a single hit
- The CycloBranch output report must correspond to the report format shown in Additional file 1: Fig. S13
- The search parameters in the report must reflect 5 ppm precursor and fragment ion m/z error tolerances, minimum relative peak intensity threshold of zero, and b-/a-ions with at most one H2O or NH3 neutral loss

## Limitations

- PK siderophores contain uncommon building blocks that turn over the orientation of peptide bonds in molecules (e.g., desferrioxamines or ornibactins), further complicating mass spectra interpretation
- Automated interpretation of NRP tandem mass spectra remains challenging due to the complex structures of the molecules
- Building the databases directly in CycloBranch instead of MassSpecBlocks requires a lot of manual labor (searching structures, manually calculating residue formulas, typing acronyms, formulas, and sequences)
- Restricting the search to b-ions and a-ions with at most one neutral loss of H2O or NH3 was done to unify configuration with NRPro, so other ion series/losses are not covered

## Evidence

- [results] 5 ppm configuration benchmark: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] ion series restriction: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [results] desferrioxamine B single hit: [evidence span withheld — anchor status: not_found]
- [abstract] database export: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [readme] tool purpose: "The application provides an export format for open source program CycloBranch (https://ms.biomed.cas.cz/cyclobranch/docs/html/) from Jiří Novák"
