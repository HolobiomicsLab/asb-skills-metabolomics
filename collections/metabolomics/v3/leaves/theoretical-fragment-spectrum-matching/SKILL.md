---
name: theoretical-fragment-spectrum-matching
description: Use when you have experimental MS/MS spectra (e.g. CID or HCD, positive
  mode) of candidate NRP/PK compounds and building-block sequences exported from MassSpecBlocks,
  and you need to identify the compound whose theoretical b-/a-ion fragment pattern
  matches the spectrum.
when_to_use_negative:
- The spectra are of ordinary peptides and you only need standard search — NRPro or
  Dereplicator may suffice, since MassSpecBlocks/CycloBranch exist for custom NRP/PK
  sequences that those tools cannot handle
- You need modification-tolerant search of spectra against a broad public spectral
  database (GNPS) rather than a custom sequence database of a few dozen candidate
  compounds
- The building blocks of your molecules cannot be decomposed from SMILES (e.g. sequences
  with flipped peptide-bond orientation in PK siderophores) — MassSpecBlocks first
  determines the block structure before matching is possible
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: MassSpecBlocks
  role: Builds the custom NRP/PK sequence and building-block database from SMILES
    structures and exports it into plain text format for CycloBranch
  repo: https://github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Performs the theoretical fragment spectrum matching (comparator) of experimental
    CID/HCD MS/MS spectra against the exported sequence database
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: NRPro
  role: Alternative comparator used as the configuration reference — matching was
    unified with NRPro by restricting to b-ions and a-ions with one neutral loss of
    H2O or NH3
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:c3d3dde89cac44d9aa51a58862251b1ffdeb52b6e7d04338950e38c8ec34cc00
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
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# theoretical-fragment-spectrum-matching

## Summary

Match experimental tandem MS/MS spectra of linear, cyclic, branched, and branch-cyclic nonribosomal peptides and polyketide siderophores against theoretical fragment spectra generated from building-block sequences exported from MassSpecBlocks, using CycloBranch's comparator for dereplication.

## When to use

Use when you have experimental MS/MS spectra (e.g. CID or HCD, positive mode) of candidate NRP/PK compounds and building-block sequences exported from MassSpecBlocks, and you need to identify the compound whose theoretical b-/a-ion fragment pattern best matches the spectrum.

## When NOT to use

- The spectra are of ordinary peptides and you only need standard search — NRPro or Dereplicator may suffice, since MassSpecBlocks/CycloBranch exist for custom NRP/PK sequences that those tools cannot handle
- You need modification-tolerant search of spectra against a broad public spectral database (GNPS) rather than a custom sequence database of a few dozen candidate compounds
- The building blocks of your molecules cannot be decomposed from SMILES (e.g. sequences with flipped peptide-bond orientation in PK siderophores) — MassSpecBlocks first determines the block structure before matching is possible

## Inputs

- Experimental tandem mass spectra (CID and HCD) of candidate compounds
- Plain-text sequence database of building-block sequences exported from MassSpecBlocks
- Precursor m/z values for each spectrum

## Outputs

- Top-scoring compound match per spectrum
- Weighted ratio of matched peaks for the top hit
- Per-mode (CID, HCD) comparator accuracy values
- CSV of comparator accuracy per fragmentation mode (e.g. comparator_accuracy_cid_hcd.csv)

## How to apply

Export the building-block sequences and building blocks of your candidate compounds from MassSpecBlocks into the plain text format accepted by CycloBranch, then load that sequence database alongside your experimental tandem mass spectra into CycloBranch. Configure the comparator with precursor and fragment ion m/z error tolerances of 5 ppm, minimum relative peak intensity threshold of zero, and restrict the search to b-ions and a-ions with at most one neutral loss of H2O or NH3 so the configuration is unified with NRPro. Enable the precursor mass filter to narrow hits by precursor m/z before fragment scoring. Execute the comparator separately for each fragmentation mode (CID and HCD), record the top-scoring compound for each spectrum, and compute per-mode accuracy as the fraction of spectra whose top hit is the correct compound. The weighted ratio of matched peaks (e.g. 96.4% for desferrioxamine B) serves as the match quality metric.

## Related tools

- **MassSpecBlocks** (Builds the custom NRP/PK sequence and building-block database from SMILES structures and exports it into plain text format for CycloBranch) — https://github.com/privrja/MassSpecBlocks
- **CycloBranch** (Performs the theoretical fragment spectrum matching (comparator) of experimental CID/HCD MS/MS spectra against the exported sequence database) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **NRPro** (Alternative comparator used as the configuration reference — matching was unified with NRPro by restricting to b-ions and a-ions with one neutral loss of H2O or NH3)

## Examples

```
MassSpecBlocks export → sequences.txt (plain text for CycloBranch) → CycloBranch comparator: load sequences.txt + 88 CID/HCD spectra; precursor & fragment error 5 ppm; intensity threshold 0; b-ions & a-ions only; ≤1 neutral loss (H2O, NH3)
```

## Evaluation signals

- Top hit for each spectrum equals the correct compound — accuracy is the fraction of correct top hits per fragmentation mode
- Precursor mass filter is enabled and the reported hit list collapses to a single correct compound (e.g. desferrioxamine B) with weighted ratio of matched peaks ≥ 96%
- Computed CID and HCD accuracies match the benchmark accuracies reported in the paper's evaluation
- Error tolerance and ion-type settings reproduce the unified configuration: 5 ppm precursor and fragment error, intensity threshold 0, only b-ions and a-ions, at most one neutral loss of H2O or NH3

## Limitations

- Users cannot create custom databases of building blocks and NRP sequences in NRPro, so matching against custom compound sets must go through MassSpecBlocks + CycloBranch instead
- PK siderophores contain uncommon building blocks that reverse peptide-bond orientation, complicating both block decomposition and fragment matching
- The comparator was configured only for b-ions, a-ions, and one neutral loss (H2O or NH3) to unify with NRPro; other fragment types and neutral losses are excluded
- It is not always easy to export flawless sequences of building blocks for tandem mass spectra analysis from source databases like Norine

## Evidence

- [results] Precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da.: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro."
- [results] Minimum threshold of relative peak intensity set to zero: "we set up the minimum threshold of relative peak intensity to zero."
- [results] Only b-ions and a-ions searched, allowing only one neutral loss of H2O or NH3: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [abstract] Export sequences and building blocks into plain text format for CycloBranch: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [intro] Comparators are used for comparing experimental tandem mass spectra with theoretical spectra.: "compares the input experimental spectra with theoretical spectra generated from the sequences of building blocks"
- [intro] NRPro users cannot create custom databases of building blocks and NRP sequences: "However, users cannot create custom databases of building blocks and NRP sequences."
- [readme] Repo documentation for the export target: "The application provides an export format for open source program CycloBranch"
