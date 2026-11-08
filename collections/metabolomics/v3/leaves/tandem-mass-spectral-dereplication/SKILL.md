---
name: tandem-mass-spectral-dereplication
description: Use when experimental tandem mass spectra of natural products (e.g. NRPs/PK
  siderophores like desferrioxamine B) must be matched against a custom in-silico
  fragment database to identify the compound producing each spectrum.
when_to_use_negative:
- When the input is already an annotated compound assignment and no spectral comparison
  against a theoretical database is needed.
- When the target molecules are not linear, cyclic, branched, or branch-cyclic nonribosomal
  peptides/polyketides representable as building-block sequences in CycloBranch.
- When custom databases of building blocks and NRP sequences cannot be created (e.g.
  NRPro cannot create custom databases).
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MassSpecBlocks
  role: Converts SMILES chemical structures of candidate NRPs/PKs into sequences of
    building blocks and exports them for CycloBranch
  repo: https://github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Loads the exported plain text sequence database and experimental tandem spectra;
    runs the comparator search with configured error tolerances to identify compounds
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: NRPro
  role: Alternative tool that compares experimental spectra with theoretical spectra
    from building-block sequences; used to unify search configuration (b-ions, a-ions,
    one neutral loss of H2O or NH3)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:e5f30c42edcebe675da2a442d41303b70c2c2b7210bf4e7c8052528240c0880c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/MassSpecBlocks
  tools:
  - CycloBranch
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# tandem-mass-spectral-dereplication

## Summary

Dereplicate unknown nonribosomal peptide and polyketide spectra by exporting a custom sequence/building-block database from MassSpecBlocks into CycloBranch and searching experimental tandem spectra against theoretically derived fragment lists.

## When to use

Use when experimental tandem mass spectra of natural products (e.g. NRPs/PK siderophores like desferrioxamine B) must be matched against a custom in-silico fragment database to identify the compound producing each spectrum.

## When NOT to use

- When the input is already an annotated compound assignment and no spectral comparison against a theoretical database is needed.
- When the target molecules are not linear, cyclic, branched, or branch-cyclic nonribosomal peptides/polyketides representable as building-block sequences in CycloBranch.
- When custom databases of building blocks and NRP sequences cannot be created (e.g. NRPro cannot create custom databases).

## Inputs

- Sequence and building-block database exported from MassSpecBlocks in CycloBranch plain text format
- Experimental tandem mass spectra (e.g. CID and HCD acquired spectra per compound)
- Search configuration: 5 ppm precursor and fragment ion m/z tolerances, b-ions/a-ions only, max one neutral loss of H2O or NH3

## Outputs

- Top-scoring compound match per input tandem spectrum
- Per-fragmentation-mode dereplication accuracy (fraction of spectra whose top hit is the correct compound), e.g. comparator_accuracy_cid_hcd.csv
- Output report with weighted ratio of matched peaks for the top hit (e.g. 96.4% for desferrioxamine B)

## How to apply

Export the sequences and building blocks of candidate compounds from MassSpecBlocks (or the MassSpecBlocks container) into the plain text format accepted by CycloBranch. Load the exported sequence database together with the experimental tandem mass spectra (e.g. 44 compounds acquired under both CID and HCD) into CycloBranch. Configure the comparator search with precursor and fragment ion m/z error tolerances of 5 ppm, minimum relative peak intensity threshold of zero, and restrict the search to b-ions and a-ions allowing only one neutral loss of H2O or NH3 so the configuration is unified with NRPro. Run the comparator separately per fragmentation mode, recording the top-scoring compound hit for each spectrum, and compute dereplication accuracy per mode as the fraction of spectra whose top hit is the correct compound.

## Related tools

- **MassSpecBlocks** (Converts SMILES chemical structures of candidate NRPs/PKs into sequences of building blocks and exports them for CycloBranch) — https://github.com/privrja/MassSpecBlocks
- **CycloBranch** (Loads the exported plain text sequence database and experimental tandem spectra; runs the comparator search with configured error tolerances to identify compounds) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **NRPro** (Alternative tool that compares experimental spectra with theoretical spectra from building-block sequences; used to unify search configuration (b-ions, a-ions, one neutral loss of H2O or NH3))

## Examples

```
1. Export sequences and blocks from MassSpecBlocks container to CycloBranch plain text format. 2. Load the exported database plus 88 tandem spectra (44 compounds × CID/HCD) into CycloBranch. 3. Comparator search with 5 ppm precursor/fragment ion tolerances, min relative intensity 0, b-ions/a-ions, max one neutral loss of H2O or NH3; run per fragmentation mode and compute accuracy per mode.
```

## Evaluation signals

- Top hit corresponds to the correct compound for each spectrum; per-mode dereplication accuracy matches the benchmark (e.g. correct desferrioxamine B identification).
- Weighted ratio of matched peaks for the top hit reaches a high fraction (e.g. 96.4% for the desferrioxamine B top hit).
- Precursor mass filter, when enabled, leaves the correct compound as the single hit.
- Computed per-mode accuracies (CID and HCD) match accuracies reported in the paper's evaluation, as recorded in comparator_accuracy_cid_hcd.csv.
- Search parameters reproduce the paper's configuration: 5 ppm precursor and fragment ion tolerances, minimum relative peak intensity of zero, only b-ions and a-ions with at most one neutral loss of H2O or NH3.

## Limitations

- The workflow is limited to compounds whose building-block sequences can be encoded in CycloBranch (NRPs and PKs); non-peptidic natural products are better served by Dereplicator+.
- PK siderophores contain uncommon building blocks that reverse the orientation of peptide bonds (e.g. desferrioxamines or ornibactins), further complicating mass spectra interpretation.
- It is not always easy to export flawless sequences of building blocks for tandem mass spectra analysis (e.g. from Norine).
- The peptide bond connecting Ac to Ile on the branch was not automatically marked because the resulting Ac block would contain only three nonhydrogen atoms, so manual curation may be needed.

## Evidence

- [abstract] Export sequences and blocks to CycloBranch plain text format: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [results] CycloBranch comparator search parameters: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Fragment ion types and neutral losses: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [results] Desferrioxamine B correct identification: [evidence span withheld — anchor status: not_found]
- [abstract] MassSpecBlocks as a web-based tool for building block sequences: "we introduce MassSpecBlocks - an open-source and web-based tool that converts the input chemical structures in SMILES format into sequences of building blocks"
- [intro] CycloBranch scope for NRP/PK spectra annotation: "In 2015, we released CycloBranch – an open-source, cross-platform, and stand-alone tool implemented in C++ and dedicated to de novo analysis and dereplication"
- [readme] MassSpecBlocks export and CycloBranch usage in README: "The application provides an export format for open source program [CycloBranch](https://ms.biomed.cas.cz/cyclobranch/docs/html/) from Jiří Novák (Laboratory of Molecular Structure Characterization -"
