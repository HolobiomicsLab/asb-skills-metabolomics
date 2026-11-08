---
name: chemical-class-grouping-analysis
description: Use when an aligned annotated molecular formula table (from IDSL.UFA)
  contains many related formulas and the goal is to collapse them into homologous
  formula-class clusters (e.g., perfluorinated, halogenated, or lipid chain families)
  rather than treating each peak independently.
when_to_use_negative:
- Input is a raw mzML/mzXML/netCDF file or a peaklist without assigned molecular formulas
  — class grouping requires formulas as input, not raw MS1 spectra
- The analysis goal is per-peak molecular formula assignment (candidate ranking against
  theoretical isotopic profiles) — that is the IDSL.UFA annotation step, not the downstream
  class-grouping step
- The chemical space contains only isolated, non-homologous compounds with no constant
  ΔH/ΔC progressions or shared fixed-carbon Σ(H+halogen) patterns — grouping will
  yield mostly ungrouped formulas and adds little interpretive value
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IDSL.UFA
  role: Provides the detect_formula_sets function that performs the ΔH/ΔC chain-progression
    and fixed-carbon Σ(H+halogen) class-grouping of annotated formulas
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Upstream tool that generates the individual and aligned peaklists (m/z-RT
    chromatographic information) which IDSL.UFA annotates; its aligned peak table
    is the substrate the class grouping is applied to
  repo: github.com/idslme/IDSL.IPA
- name: R
  role: 'Execution environment: grouping is run as an R package function on the aligned
    annotated molecular formula table'
  repo: https://cran.r-project.org/package=IDSL.UFA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:2d706a6aa43f0fec217fc807c45fa523627bae7b7941250a8be27e155cf5afa6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chemical-class-grouping-analysis

## Summary

Groups annotated molecular formulas into chemical-class sub-structure clusters using IDSL.UFA's class-detection rules: chain progressions defined by constant ΔH/ΔC ratios (2 for polymeric, 1/2 for cyclic chains) and fixed-carbon classes with a constant sum of H plus halogens (Br, Cl, F, I), as used to recover PCB and PBDE homologous series.

## When to use

Use when an aligned annotated molecular formula table (from IDSL.UFA) contains many related formulas and the goal is to collapse them into homologous formula-class clusters (e.g., perfluorinated, halogenated, or lipid chain families) rather than treating each peak independently.

## When NOT to use

- Input is a raw mzML/mzXML/netCDF file or a peaklist without assigned molecular formulas — class grouping requires formulas as input, not raw MS1 spectra
- The analysis goal is per-peak molecular formula assignment (candidate ranking against theoretical isotopic profiles) — that is the IDSL.UFA annotation step, not the downstream class-grouping step
- The chemical space contains only isolated, non-homologous compounds with no constant ΔH/ΔC progressions or shared fixed-carbon Σ(H+halogen) patterns — grouping will yield mostly ungrouped formulas and adds little interpretive value

## Inputs

- Aligned annotated molecular formula table (CSV) output by IDSL.UFA, with one row per peak/formula and element counts derivable from the formula string
- Element-count table with columns for C, H, and halogens (Br, Cl, F, I) — parsed from the annotated formulas

## Outputs

- formula_class_table.csv — table mapping each input formula to its detected class cluster (or ungrouped status)
- Distinct formula-class clusters, each representing a homologous sub-structure family (e.g., a PCB congener ladder, a PBDE homolog series, a polymeric chain progression)

## How to apply

Parse each formula in the aligned annotated molecular formula table into element counts (C, H, Br, Cl, F, I, N, O, P, S, etc.), then run IDSL.UFA's detect_formula_sets grouping algorithm. Two grouping rules are applied: (1) chain-progression classes where consecutive formulas in a series differ by a constant ΔH/ΔC ratio — ΔH/ΔC = 2 identifies polymeric chains and ΔH/ΔC = 1/2 identifies cyclic chains; (2) fixed-carbon classes where formulas sharing the same carbon count are grouped when they share a constant value of Σ(H + Br + Cl + F + I), which captures homologous substitution patterns like PCBs (fixed C + Σ(H+Cl) ladders) and PBDEs. Each formula is assigned to at most one class cluster or left ungrouped; cluster membership per formula is the primary output. The article reports this step revealed 7,615, 18,452, and 32,107 distinct formula classes across studies, so expected cluster counts scale with study size and chemical space.

## Related tools

- **IDSL.UFA** (Provides the detect_formula_sets function that performs the ΔH/ΔC chain-progression and fixed-carbon Σ(H+halogen) class-grouping of annotated formulas) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Upstream tool that generates the individual and aligned peaklists (m/z-RT chromatographic information) which IDSL.UFA annotates; its aligned peak table is the substrate the class grouping is applied to) — github.com/idslme/IDSL.IPA
- **R** (Execution environment: grouping is run as an R package function on the aligned annotated molecular formula table) — https://cran.r-project.org/package=IDSL.UFA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- formula_class_table.csv exists, is a parseable table, and every input formula appears exactly once (no duplicate formula entries)
- Every input formula is assigned to exactly one class cluster or is explicitly left ungrouped — no formula is dropped or double-assigned
- Within each detected cluster, the invariant holds: consecutive members satisfy the claimed ΔH/ΔC ratio (2 or 1/2), or all members share the same carbon count and the same Σ(H + Br + Cl + F + I)
- Cluster count is plausible relative to study size: the article reported 7,615, 18,452, and 32,107 distinct formula classes for large exposomics studies, so counts far outside that order of magnitude for similar data warrant re-inspection of parameters
- Spot-check known homologous families: halogenated classes like PCBs and PBDEs should be recovered as fixed-carbon clusters with constant Σ(H+Cl) or Σ(H+Br), respectively

## Limitations

- Grouping depends entirely on the quality of upstream formula annotation — false-positive formula assignments propagate into spurious class clusters
- The ΔH/ΔC = 2 and 1/2 rules and the fixed-carbon Σ(H+halogen) rule are fixed in detect_formula_sets; chemical classes that do not fit these two canonical sub-structure patterns will not be detected
- Only a small subset of elements (C, H, Br, Cl, F, I) participates in the class sub-structure definitions used here; formulas differing in heteroatom counts (N, O, P, S) within the same class are still grouped, which may conflate distinct sub-structures
- Class detection is a grouping heuristic, not a structural assignment — clusters identify candidate sub-structures (e.g., Cl-PFTrEAs) but do not confirm structures without MS2 or database corroboration

## Evidence

- [methods] detect_formula_sets grouping rules: [evidence span withheld — anchor status: not_found]
- [results] class detection highlighted distinct formula classes: "Grouping these formulas by a class detection approach (see method) highlighted that 7,615, 18,452, and 32,107 distinct formula classes"
- [results] aligned annotated peak table summarization: "summarizes the formula annotations using detection frequency and median ranks in multiple samples (aligned annotated peak table)"
- [readme] README: UFA requires IPA chromatographic information: [evidence span withheld — anchor status: not_found]
- [readme] README: IPDB generation feature: [evidence span withheld — anchor status: external_doc]
- [readme] README: UFAx elemental composition scope: [evidence span withheld — anchor status: external_doc]
