---
name: lipid-structure-maximum-common-subgraph-comparison
description: Use when pairwise structural similarity between lipid species must be
  quantified from molecular graphs — e.g. building a lipid distance matrix, constructing
  a lipid space model, or testing whether counting double-bond positions changes the
  overlap between two fatty acids.
when_to_use_negative:
- The task requires combining lipid abundances with structural similarity — the raw
  MCS comparison is purely structural and does not add quantitative information, which
  the article notes makes models heavily influenced by low-abundant species.
- Lipid species are only compared by exact name/identifier matches and no molecular
  structures are available, so no molecular graph can be built.
- An unbounded (0-to-infinity) distance is required for downstream visualization;
  the default bound metric ranges 0–1 and switching modes trades lipid space visibility
  for accuracy.
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: LipidSpace
  role: Standalone workstation application that builds molecular graphs for lipids,
    runs the LaWeCSE-based MCS computation with Jaccard scoring, and exposes the DB-position
    and sn-position comparison modes
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid shorthand names (e.g. FA 18:2(9Z,12Z), PC 18:0_16:1)
    before graph construction
  repo: https://github.com/lifs-tools/cppgoslin
- name: LIPID MAPS
  role: Source database from which lipid structures are extracted at the highest structural
    resolution level for graph construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:0d1b328967db784c52a8685b4766c05a21adcd7e59ac787c14ee168567052b97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LIPID MAPS
  - LipidSpace
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipid-structure-maximum-common-subgraph-comparison

## Summary

Computes pairwise structural similarity between lipid species by representing each lipid as a molecular graph (atoms and bonds as components) and applying a LaWeCSE-based maximum common subgraph (MCS) approach scored with the Jaccard index. Used to build the structural distance matrix underlying LipidSpace's lipid space model and to verify that double-bond (DB) position comparisons are handled correctly.

## When to use

Use when pairwise structural similarity between lipid species must be quantified from molecular graphs — e.g. building a lipid distance matrix, constructing a lipid space model, or testing whether counting double-bond positions changes the overlap between two fatty acids.

## When NOT to use

- The task requires combining lipid abundances with structural similarity — the raw MCS comparison is purely structural and does not add quantitative information, which the article notes makes models heavily influenced by low-abundant species.
- Lipid species are only compared by exact name/identifier matches and no molecular structures are available, so no molecular graph can be built.
- An unbounded (0-to-infinity) distance is required for downstream visualization; the default bound metric ranges 0–1 and switching modes trades lipid space visibility for accuracy.

## Inputs

- Lipid species shorthand names (flat or pivot csv/xlsx tables, or mzTab-M files), parsed and standardized by Goslin
- Molecular structure records for individual lipids (e.g. LIPID MAPS or SwissLipids entries at the highest structural resolution level)
- Optional mode flags: DB-position-aware vs DB-agnostic comparison; 'Ignore lipid sn-positions'; bound vs unbound distance metric

## Outputs

- Per-pair intersection (shared) and union component counts from the MCS computation
- A pairwise lipid species distance/similarity matrix scored via the Jaccard index, bounded between 0 (identical) and 1 in default mode
- Mode/intersection/union verification rows (e.g. written to double_bond_mcs_verification.csv)
- An n x n distance matrix feeding the PCA-based global lipid space model

## How to apply

Parse the lipid shorthand names (e.g. FA 18:2(9Z,12Z), FA 18:1(9Z)) with the Goslin parser (C++ cppgoslin library in LipidSpace) and retrieve the corresponding LIPID MAPS structure records. Build each lipid's molecular graph with atoms and bonds as components, then compute the maximum common subgraph via the LaWeCSE approach and score pairwise similarity with the Jaccard index (intersection over union of components). Count double-bond positions in the forward (carbonyl-side) direction when running the DB-position-aware mode; in the DB-agnostic mode all double-bond matches count as mismatches. For the reference pair FA 18:2(9Z,12Z) vs FA 18:1(9Z), expect 38 of 39 shared components with DB positions considered and 36 of 39 without. When lipid names do not specify sn-positions (e.g. PC 18:0_16:1), activate 'Ignore lipid sn-positions' so all FA-chain combinations are compared and the lowest distance is chosen, trading accuracy against performance.

## Related tools

- **LipidSpace** (Standalone workstation application that builds molecular graphs for lipids, runs the LaWeCSE-based MCS computation with Jaccard scoring, and exposes the DB-position and sn-position comparison modes) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid shorthand names (e.g. FA 18:2(9Z,12Z), PC 18:0_16:1) before graph construction) — https://github.com/lifs-tools/cppgoslin
- **LIPID MAPS** (Source database from which lipid structures are extracted at the highest structural resolution level for graph construction)

## Examples

```
cd cppgoslin/examples && make && ./lipid_name_parser  # parse FA 18:2(9Z,12Z) and FA 18:1(9Z) before loading them into LipidSpace for the MCS comparison
```

## Evaluation signals

- For the reference pair FA 18:2(9Z,12Z) vs FA 18:1(9Z), the DB-position-aware mode yields 38 shared components of 39; the DB-agnostic mode yields 36 of 39.
- The DB-position-aware intersection count must be greater than or equal to the DB-agnostic count for any structurally similar lipid pair, since ignoring DB positions can only convert matches into mismatches.
- Jaccard-based distances in default (bound) mode must lie in [0, 1], with 0 only for identical lipid structures.
- Both input lipid names must parse successfully with Goslin before graph construction; a parse failure invalidates the comparison.
- The resulting pairwise distance matrix is symmetric with a zero diagonal, since it feeds an n x n PCA (Lanczos method) to produce the lipid space model.

## Limitations

- When sn-positions are unspecified (e.g. PC 18:0_16:1), the all-combinations FA comparison mode gives more accurate results but decreases performance.
- In the DB-agnostic mode all double-bond matches count as mismatches, systematically underestimating overlap for lipids differing only in double-bond position.
- The default bound distance metric (0–1) may reduce accuracy compared to the unbound metric, but the unbound mode reduces visibility of the lipid spaces since distances can become very large.
- Quantitative abundance information is not part of this structural comparison, so low-abundant lipid species influence the resulting structural space models.

## Evidence

- [methods] we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15: "we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15"
- [methods] It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [methods] Correctness of Double Bond Comparisons: "Correctness of Double Bond Comparisons"
- [readme] A graph-based comparison of lipid structures allows to calculate distances between lipids and to determine similarities across lipidomes.: "A graph-based comparison of lipid structures allows to calculate distances between lipids and to determine similarities across lipidomes."
- [readme] when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
