---
name: cid-hcd-fragmentation-accuracy-comparison
description: Use when you have matched sets of CID and HCD tandem MS spectra of the
  same compounds (e.g., 44 CID + 44 HCD NRP/PK spectra) and need to measure or compare
  how the fragmentation method changes annotation/dereplication tool accuracy.
when_to_use_negative:
- When spectra come from a single fragmentation condition only — there is no CID vs
  HCD contrast to compute and a single-condition accuracy evaluation is the correct
  skill
- When you need de novo sequencing rather than annotation/dereplication accuracy against
  known references — this skill scores tool reports against known compound identities
- When input spectra are low-resolution or the tool tolerances must be expressed in
  Da rather than ppm, since the 5 ppm configuration is defined for high-resolution
  data
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CycloBranch
  role: Annotation engine run separately over CID and HCD spectrum sets against the
    MassSpecBlocks-exported sequence database; reports top hits with weighted ratio
    of matched peaks
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: MassSpecBlocks
  role: Builds the custom NRP/PK sequence and building block database and exports
    it to the plain text format CycloBranch consumes
  repo: github.com/privrja/MassSpecBlocks
- name: NRPro
  role: Comparison tool run at the same 5 ppm tolerances to benchmark annotation counts
    across fragmentation conditions
- name: Dereplicator
  role: Comparison tool run at 0.005 Da tolerances; annotates far fewer peaks (e.g.,
    6/29 on pseudacyclin A), used as a lower-performing baseline in the CID vs HCD
    comparison
- name: Dereplicator+
  role: Comparison tool run at 0.005 Da tolerances; extends annotation to non-peptidic
    natural products and is benchmarked in the per-condition comparison
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:26e599b4b245778ff02a283bdf2072a4d2b2af1bf8352c71b3b0911e67203c52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - Dereplicator
  - Dereplicator+
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cid-hcd-fragmentation-accuracy-comparison

## Summary

A method for quantifying how collision-induced dissociation (CID) versus higher-energy C-trap dissociation (HCD) affects peptide-spectrum annotation accuracy, by comparing per-condition Annotator/Comparator accuracies of tools like CycloBranch, NRPro, Dereplicator, and Dereplicator+ on NRP/PK siderophore tandem mass spectra.

## When to use

Use when you have matched sets of CID and HCD tandem MS spectra of the same compounds (e.g., 44 CID + 44 HCD NRP/PK spectra) and need to measure or compare how the fragmentation method changes annotation/dereplication tool accuracy.

## When NOT to use

- When spectra come from a single fragmentation condition only — there is no CID vs HCD contrast to compute and a single-condition accuracy evaluation is the correct skill
- When you need de novo sequencing rather than annotation/dereplication accuracy against known references — this skill scores tool reports against known compound identities
- When input spectra are low-resolution or the tool tolerances must be expressed in Da rather than ppm, since the 5 ppm configuration is defined for high-resolution data

## Inputs

- Matched sets of high-resolution tandem mass spectra per fragmentation condition (44 CID and 44 HCD spectra of NRP/PK compounds)
- Reference compound identities for each spectrum (ground-truth annotation targets)
- Sequence and building block databases exported from MassSpecBlocks into CycloBranch's plain text format
- Reproduced or reported per-condition accuracy tables (Annotator and Comparator accuracies per tool per condition)

## Outputs

- cid_hcd_accuracy_comparison.csv with columns: tool, CID accuracy, HCD accuracy, absolute difference, relative difference
- annotator_accuracy_by_condition.csv with per-condition annotation accuracy values
- Per-spectrum peak annotation counts and weighted ratios of matched peak intensities (e.g., 26/29 peaks, 99% for pseudacyclin A)

## How to apply

Run the annotation tool (e.g., CycloBranch against a MassSpecBlocks-built sequence/building block database) separately over the CID spectrum set and the HCD spectrum set, keeping parameters identical across conditions: precursor and fragment ion m/z error tolerances of 5 ppm, minimum relative peak intensity threshold of zero, and searching only b-ions and a-ions with at most one neutral loss of H2O or NH3. Compute annotation accuracy per condition by comparing each returned report against the reference compound identity, or count annotated experimental peaks per spectrum (e.g., CycloBranch annotated 26 of 29 peaks of pseudacyclin A at a 99% weighted ratio of matched peak intensities). Parse the per-tool, per-condition accuracy values into a tidy pandas dataframe and compute absolute (CID − HCD) and relative ((CID − HCD)/HCD) accuracy deltas. Assemble a comparison table with columns tool, CID accuracy, HCD accuracy, absolute difference, relative difference and write it to cid_hcd_accuracy_comparison.csv. Note that when including Dereplicator/Dereplicator+, their 0.005 Da tolerances (vs 5 ppm for CycloBranch/NRPro) are the tools' own defaults and must be recorded so cross-tool deltas are interpreted with the tolerance difference in mind.

## Related tools

- **CycloBranch** (Annotation engine run separately over CID and HCD spectrum sets against the MassSpecBlocks-exported sequence database; reports top hits with weighted ratio of matched peaks) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **MassSpecBlocks** (Builds the custom NRP/PK sequence and building block database and exports it to the plain text format CycloBranch consumes) — github.com/privrja/MassSpecBlocks
- **NRPro** (Comparison tool run at the same 5 ppm tolerances to benchmark annotation counts across fragmentation conditions)
- **Dereplicator** (Comparison tool run at 0.005 Da tolerances; annotates far fewer peaks (e.g., 6/29 on pseudacyclin A), used as a lower-performing baseline in the CID vs HCD comparison)
- **Dereplicator+** (Comparison tool run at 0.005 Da tolerances; extends annotation to non-peptidic natural products and is benchmarked in the per-condition comparison)

## Examples

```
import pandas as pd; df = pd.DataFrame(rows, columns=['tool','CID accuracy','HCD accuracy','absolute difference','relative difference']); df.to_csv('cid_hcd_accuracy_comparison.csv', index=False)
```

## Evaluation signals

- Recompute every accuracy cell and delta from the source per-condition tables; the comparison table is accepted only if all recomputed values match exactly
- Computed CID and HCD accuracies must agree with the Annotator accuracy values reported in the paper's results section
- Per-spectrum sanity anchors: CycloBranch should report a 96.4% weighted ratio on desferrioxamine B, 26/29 peaks (99%) on pseudacyclin A, and 21/25 peaks (97%) on roseotoxin A
- Output CSV schema check: rows for each tool with the exact columns tool, CID accuracy, HCD accuracy, absolute difference, relative difference
- Confirm both fragmentation conditions used identical annotation parameters (5 ppm, threshold zero, b/a-ions with one H2O or NH3 neutral loss) so deltas reflect fragmentation, not configuration

## Limitations

- Cross-tool accuracy deltas are confounded by tolerance units: CycloBranch and NRPro used 5 ppm while Dereplicator and Dereplicator+ used 0.005 Da
- MassSpecBlocks did not automatically mark the peptide bond connecting Ac to Ile on the branch because the resulting Ac block would contain only three nonhydrogen atoms, so some branch structures may need manual correction before annotation
- PK siderophores contain uncommon building blocks that turn over the orientation of peptide bonds (e.g., desferrioxamines, ornibactins), complicating mass spectra interpretation and potentially depressing accuracy for both conditions
- A precursor mass filter can materially change the output report (shown enabled vs disabled for desferrioxamine B), so filter state must be held constant across conditions

## Evidence

- [results] Fragmentation tolerance asymmetry: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Peak intensity threshold and ion series configuration: "we set up the minimum threshold of relative peak intensity to zero."
- [results] Ion-series restriction rationale: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [abstract] Database construction feeding the annotation runs: "we introduce MassSpecBlocks - an open-source and web-based tool that converts the input chemical structures in SMILES format into sequences of building blocks"
- [readme] Export path from MassSpecBlocks to CycloBranch: "The application provides an export format for open source program [CycloBranch](https://ms.biomed.cas.cz/cyclobranch/docs/html/)"
