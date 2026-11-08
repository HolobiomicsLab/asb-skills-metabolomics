---
name: cyclobranch-annotator-comparator-metrics-derivation
description: Use when benchmarking NRP/PK tandem-MS annotation tools (CycloBranch,
  NRPro, Dereplicator, Dereplicator+) across CID vs HCD conditions and you need per-tool
  accuracy deltas, e.g. weighted peak-matched intensity ratios.
when_to_use_negative:
- Input spectra are low-resolution or the goal is de novo sequencing rather than dereplication-style
  annotation against a known compound database
- You need custom building-block/sequence databases within NRPro — the article states
  'users cannot create custom databases of building blocks and NRP sequences'
- Comparing against tools without an analogous weighted matched-intensity metric —
  raw annotated-peak counts (e.g. Dereplicator 6/29 on pseudacyclin A) are not directly
  comparable to CycloBranch weighted ratios
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CycloBranch
  role: Annotates linear, cyclic, branched, and branch-cyclic NRP/PK tandem MS spectra
    and reports weighted ratio of matched peak intensities; source of Annotator/Comparator
    metrics
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: MassSpecBlocks
  role: Converts SMILES structures into building-block sequences and exports plain-text
    databases consumed by CycloBranch for the annotation run
  repo: https://github.com/privrja/MassSpecBlocks
- name: NRPro
  role: Comparison tool whose configuration (5 ppm tolerances, b- and a-ions, one
    neutral loss of H2O or NH3) the CycloBranch setup is unified with
- name: Dereplicator
  role: Baseline annotator run at 0.005 Da tolerance on the same CID/HCD sets
- name: Dereplicator+
  role: Baseline annotator for non-peptidic natural products, run at 0.005 Da tolerance
    on the same CID/HCD sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:f36686a9309b4f76bbb63fe337f0d0f1b54f78df5bc2a30f1a6608f9e1f65be8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - Dereplicator
  - Dereplicator+
  - MassSpecBlocks
  - NRPro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cyclobranch-annotator-comparator-metrics-derivation

## Summary

Derives and compares per-tool Annotator and Comparator accuracy metrics for CycloBranch (and NRPro/Dereplicator/Dereplicator+) on high-resolution CID vs HCD tandem MS spectra of NRPs and PK siderophores, producing a validated tool-by-condition accuracy delta table.

## When to use

Use when benchmarking NRP/PK tandem-MS annotation tools (CycloBranch, NRPro, Dereplicator, Dereplicator+) across CID vs HCD conditions and you need per-tool accuracy deltas, e.g. weighted peak-matched intensity ratios.

## When NOT to use

- Input spectra are low-resolution or the goal is de novo sequencing rather than dereplication-style annotation against a known compound database
- You need custom building-block/sequence databases within NRPro — the article states 'users cannot create custom databases of building blocks and NRP sequences'
- Comparing against tools without an analogous weighted matched-intensity metric — raw annotated-peak counts (e.g. Dereplicator 6/29 on pseudacyclin A) are not directly comparable to CycloBranch weighted ratios

## Inputs

- 44-spectrum CID tandem MS set of NRPs and PK siderophores
- 44-spectrum HCD tandem MS set of NRPs and PK siderophores
- MassSpecBlocks-built CycloBranch building-block and sequence databases (plain-text export format)
- Reproduced per-condition accuracy tables (Annotator and Comparator accuracies per tool)

## Outputs

- cid_hcd_accuracy_comparison.csv with columns tool, CID accuracy, HCD accuracy, absolute difference, relative difference
- Tidy dataframe of per-tool per-condition Annotator/Comparator accuracies

## How to apply

Run annotation of the 44-spectrum CID set and 44-spectrum HCD set with CycloBranch using MassSpecBlocks-built sequence/building-block databases, setting precursor and fragment ion m/z error tolerances to 5 ppm for CycloBranch and NRPro (0.005 Da for Dereplicator and Dereplicator+), the minimum relative peak intensity threshold to zero, and searching only b-ions and a-ions with at most one neutral loss of H2O or NH3 to unify configuration with NRPro. Parse Annotator and Comparator accuracy values per tool per fragmentation condition into a tidy pandas dataframe. Compute per-tool absolute deltas (CID accuracy minus HCD accuracy) and relative deltas ((CID − HCD)/HCD), then assemble a comparison table with columns tool, CID accuracy, HCD accuracy, absolute difference, and relative difference and write it to cid_hcd_accuracy_comparison.csv. The rationale is that CycloBranch reports the weighted ratio of matched peak intensities per hit (e.g. 99% on pseudacyclin A, 97% on roseotoxin A), which captures annotation completeness better than raw peak counts alone.

## Related tools

- **CycloBranch** (Annotates linear, cyclic, branched, and branch-cyclic NRP/PK tandem MS spectra and reports weighted ratio of matched peak intensities; source of Annotator/Comparator metrics) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **MassSpecBlocks** (Converts SMILES structures into building-block sequences and exports plain-text databases consumed by CycloBranch for the annotation run) — https://github.com/privrja/MassSpecBlocks
- **NRPro** (Comparison tool whose configuration (5 ppm tolerances, b- and a-ions, one neutral loss of H2O or NH3) the CycloBranch setup is unified with)
- **Dereplicator** (Baseline annotator run at 0.005 Da tolerance on the same CID/HCD sets)
- **Dereplicator+** (Baseline annotator for non-peptidic natural products, run at 0.005 Da tolerance on the same CID/HCD sets)

## Examples

```
import pandas as pd; df=pd.read_csv('tidy.csv'); df['abs_delta']=df.cid-df.hcd; df['rel_delta']=(df.cid-df.hcd)/df.hcd; df.to_csv('cid_hcd_accuracy_comparison.csv',index=False)
```

## Evaluation signals

- Recomputed accuracy cells and deltas must match the reproduced per-condition tables exactly; the comparison table is accepted only on exact match
- CycloBranch weighted ratios should reproduce the reported values: 26/29 peaks (99%) on pseudacyclin A and 21/25 peaks (97%) on roseotoxin A
- All annotation runs must use 5 ppm precursor/fragment tolerances for CycloBranch and NRPro and 0.005 Da for Dereplicator/Dereplicator+, with relative peak intensity threshold zero
- Output CSV schema must contain exactly: tool, CID accuracy, HCD accuracy, absolute difference, relative difference
- Relative deltas must be dimensionless and consistent with (CID − HCD)/HCD of the tabulated accuracies

## Limitations

- CycloBranch databases built directly in the tool require substantial manual labor (calculating residue formulas, typing acronyms, formulas, and sequences); MassSpecBlocks automates this but its automatic block detection can miss peptide bonds — e.g. the Ac–Ile branch bond was not marked because the resulting Ac block would contain only three nonhydrogen atoms
- Dereplicator and Dereplicator+ annotate far fewer experimental peaks (6/29 and 14/29 on pseudacyclin A), so cross-tool comparisons of peak counts are inherently asymmetric
- Metrics are only comparable when ion series and neutral-loss settings are unified across tools (b-ions, a-ions, at most one H2O or NH3 loss)

## Evidence

- [results] 5 ppm tolerances unify config with NRPro: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] ion series and neutral losses restricted: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [abstract] CycloBranch scope covers branched/branch-cyclic spectra: "CycloBranch is an open-source, cross-platform, and stand-alone tool that we recently released for annotating spectra of linear, cyclic, branched, and branch-cyclic nonribosomal peptides and"
- [abstract] MassSpecBlocks exports databases for CycloBranch annotation: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [results] pseudacyclin A correctly decomposed: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [readme] README confirms CycloBranch export integration: "The application provides an export format for open source program [CycloBranch](https://ms.biomed.cas.cz/cyclobranch/docs/html/) from Jiří Novák"
