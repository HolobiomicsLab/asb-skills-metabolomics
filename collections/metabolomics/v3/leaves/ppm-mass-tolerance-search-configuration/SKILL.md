---
name: ppm-mass-tolerance-search-configuration
description: Use when searching experimental high-resolution tandem MS/MS spectra
  of NRP/PK compounds against a custom building-block/sequence database (e.g. MassSpecBlocks-exported)
  and benchmarking top-1/top-10 retrieval accuracy with strict mass-accuracy configurations.
when_to_use_negative:
- When running Dereplicator or Dereplicator+, which are configured with absolute tolerances
  of 0.005 Da rather than ppm values — transplanting the 5 ppm configuration would
  be wrong
- When the goal is de novo spectral annotation in CycloBranch rather than database
  search/dereplication against a sequence database
- When a custom building-block/sequence database must be constructed in NRPro — NRPro
  users cannot create custom databases of building blocks and NRP sequences
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0157
tools:
- name: CycloBranch
  role: Runs the ppm-tolerance database search (dereplication) of tandem mass spectra
    against the imported sequence database and emits the ranked output report
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: MassSpecBlocks
  role: Creates and exports the custom NRP/PK sequence and building-block database
    in the plain text format consumed by CycloBranch
  repo: https://github.com/privrja/MassSpecBlocks
- name: NRPro
  role: Comparison tool whose 5 ppm, b/a-ion, single-neutral-loss configuration the
    CycloBranch settings are unified with
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:70511462fd23b75d175aa1ee8ffc73bc14216a75a6d84c3caa5c4529e345903d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ppm-mass-tolerance-search-configuration

## Summary

Configures CycloBranch (and comparably NRPro) dereplication/database searches of high-resolution tandem mass spectra of NRPs and polyketides using 5 ppm precursor and fragment ion m/z tolerances, zero relative peak intensity threshold, and restriction to b-ions and a-ions with at most one neutral loss of H2O or NH3.

## When to use

Use when searching experimental high-resolution tandem MS/MS spectra of NRP/PK compounds against a custom building-block/sequence database (e.g. MassSpecBlocks-exported) and benchmarking top-1/top-10 retrieval accuracy with strict mass-accuracy configurations.

## When NOT to use

- When running Dereplicator or Dereplicator+, which are configured with absolute tolerances of 0.005 Da rather than ppm values — transplanting the 5 ppm configuration would be wrong
- When the goal is de novo spectral annotation in CycloBranch rather than database search/dereplication against a sequence database
- When a custom building-block/sequence database must be constructed in NRPro — NRPro users cannot create custom databases of building blocks and NRP sequences

## Inputs

- Experimental high-resolution tandem mass spectra of NRP/PK compounds (e.g. deposited spectra such as desferrioxamine B)
- Plain-text sequence and building-block database exported from MassSpecBlocks (e.g. container 'Nonribosomal Peptides and Siderophores': 85 building blocks, 146 sequences)
- Known compound identities for the spectra (ground-truth labels for accuracy computation)

## Outputs

- CycloBranch output report (JSON/plain text) listing ranked structure identifications per spectrum
- Top-1 and top-10 database-search (dereplication) accuracy values for the 5 ppm configuration

## How to apply

Export the NRP/siderophore sequence and building-block database from MassSpecBlocks (e.g. the default container 'Nonribosomal Peptides and Siderophores' with 85 building blocks and 146 sequences) into the plain text format accepted by CycloBranch. In CycloBranch, set the precursor and fragment ion m/z error tolerances to 5 ppm (matching the configuration used for NRPro; Dereplicator/Dereplicator+ instead use 0.005 Da, so this ppm configuration is tool-specific). Set the minimum threshold of relative peak intensity to zero, and restrict the theoretical fragment search to b-ions and a-ions allowing only one neutral loss of H2O or NH3, which unifies the configuration with NRPro. Optionally enable the precursor mass filter — in the paper this turned a desferrioxamine B search against 146 sequences into a single hit. Run the search over the deposited tandem mass spectra, parse the ranked output report (JSON/plain text identifications per spectrum), and compute top-1 and top-10 accuracy against the known compound identities.

## Related tools

- **CycloBranch** (Runs the ppm-tolerance database search (dereplication) of tandem mass spectra against the imported sequence database and emits the ranked output report) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **MassSpecBlocks** (Creates and exports the custom NRP/PK sequence and building-block database in the plain text format consumed by CycloBranch) — https://github.com/privrja/MassSpecBlocks
- **NRPro** (Comparison tool whose 5 ppm, b/a-ion, single-neutral-loss configuration the CycloBranch settings are unified with)

## Evaluation signals

- With the precursor mass filter enabled, the desferrioxamine B spectrum searched against 146 NRP and siderophore sequences yields desferrioxamine B as a single hit
- Computed top-1 and top-10 accuracies match the benchmark values reported in the paper's results section for the 5 ppm CycloBranch configuration
- The output report corresponds to the report format shown in Additional file 1: Fig. S13
- Fragment ion matches are restricted to b-ions and a-ions with at most one neutral loss of H2O or NH3 (no other ion series or multiple neutral losses present)
- Precursor and fragment tolerances in the run configuration read exactly 5 ppm (not Da), with minimum relative peak intensity threshold of zero

## Limitations

- The ppm configuration is not portable to Dereplicator/Dereplicator+, which require 0.005 Da tolerances instead
- NRPro cannot accept custom databases of building blocks and NRP sequences, so the MassSpecBlocks-exported database workflow is CycloBranch-specific
- In branch-cyclic structures, some peptide bonds may not be automatically marked by MassSpecBlocks (e.g. the Ac-to-Ile branch bond, because the resulting Ac block would contain only three nonhydrogen atoms), affecting database correctness
- Enabling vs. disabling the precursor mass filter changes the report (e.g. single hit vs. broader ranking), so the filter state must be recorded when reproducing benchmarks

## Evidence

- [results] Precursor and fragment ion tolerances: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Ion series restriction: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [results] Precursor mass filter report: "In Additional file 1: Fig. S13, we show an output report if a precursor mass filter was enabled."
- [readme] Export path from MassSpecBlocks to CycloBranch: "The application provides an export format for open source program [CycloBranch]"
- [results] Intensity threshold: "we set up the minimum threshold of relative peak intensity to zero."
