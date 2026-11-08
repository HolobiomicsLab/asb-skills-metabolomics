---
name: jaccard-index-component-counting
description: Use when you need pairwise structural similarity of lipid species from
  molecular graphs, e.g., to verify MCS/Jaccard overlap counts for structurally similar
  lipids or to quantify how double-bond position information changes lipid-lipid distances.
when_to_use_negative:
- Input is already a feature/abundance table and the goal is purely quantitative comparison
  — Jaccard on molecular graphs ignores abundance, and the article notes that structural
  models are 'heavily influenced by low abundant lipid species' when quantitative
  data is mixed in elsewhere
- Lipid species names cannot be parsed to a structure (no recognizable shorthand/identifier),
  since graph construction requires a standardized parsed name via Goslin
- You only need exact lipid name/identifier matching between lipidomes — the article
  contrasts structural-similarity matching with 'only exact matches of the lipid species'
  names/identifiers'
edam_topics:
- http://edamontology.org/topic_3389
- http://edamontology.org/topic_3315
tools:
- name: LipidSpace
  role: Standalone application that builds lipid molecular graphs, computes the LaWeCSE-based
    MCS with Jaccard index, and exposes DB-position and sn-position comparison modes
  repo: https://github.com/lifs-tools/lipidspace
- name: cppgoslin (Goslin)
  role: Parses and standardizes lipid shorthand names into structures that feed the
    graph/MCS comparison
  repo: https://github.com/lifs-tools/cppgoslin
- name: lambda-lanczos
  role: 'Downstream: computes PCA via the Lanczos method on the n x n Jaccard distance
    matrix to build the lipid space model'
  repo: https://github.com/mrcdr/lambda-lanczos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:93477531aa34114e8487e626450c3f6e0dc81fe1968b01a8a44c8aa860d5f898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - LipidSpace
  - cppgoslin (Goslin)
  - lambda-lanczos
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3389
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# jaccard-index-component-counting

## Summary

Computes pairwise structural similarity between lipid species by building their molecular graphs, finding the maximum common subgraph (LaWeCSE-based MCS), and dividing shared components (atoms and bonds) by united components via the Jaccard index — with explicit support for double-bond-position-aware vs DB-agnostic comparison modes.

## When to use

Use when you need pairwise structural similarity of lipid species from molecular graphs, e.g., to verify MCS/Jaccard overlap counts for structurally similar lipids or to quantify how double-bond position information changes lipid-lipid distances.

## When NOT to use

- Input is already a feature/abundance table and the goal is purely quantitative comparison — Jaccard on molecular graphs ignores abundance, and the article notes that structural models are 'heavily influenced by low abundant lipid species' when quantitative data is mixed in elsewhere
- Lipid species names cannot be parsed to a structure (no recognizable shorthand/identifier), since graph construction requires a standardized parsed name via Goslin
- You only need exact lipid name/identifier matching between lipidomes — the article contrasts structural-similarity matching with 'only exact matches of the lipid species' names/identifiers'

## Inputs

- Lipid species names/identifiers in LIPID MAPS shorthand notation (e.g., FA 18:2(9Z,12Z), FA 18:1(9Z)) or LIPID MAPS/SwissLipids structure records
- Lipidomics data tables in csv/xlsx (flat or pivot layout) or mzTab-M format from which lipid species are drawn

## Outputs

- Pairwise MCS intersection (shared) and union (united) component counts per species pair
- Jaccard-index-based structural similarity/distance values per lipid pair (bound metric: 0 = identical)
- Mode/comparison summary rows, e.g., double_bond_mcs_verification.csv with mode, intersection, union columns

## How to apply

Parse the lipid shorthand names (e.g., FA 18:2(9Z,12Z), FA 18:1(9Z)) with the Goslin parser (cppgoslin in LipidSpace) to obtain a standardized structure representation, then build the two molecular graphs where atoms and bonds are the counting components. Compute the maximum common subgraph using LipidSpace's LaWeCSE-based MCS approach, counting double-bond positions in the forward (carbonyl-side) direction when the DB-position-aware mode is active; in the DB-agnostic mode every double-bond match counts as a mismatch. The Jaccard index is the ratio of shared (intersection) to united (union) components, yielding the pairwise structural similarity used both for single species pairs and for the n x n distance matrix feeding the lipid space model. For the FA 18:2(9Z,12Z) vs FA 18:1(9Z) reference pair, the DB-aware comparison yields 38 of 39 shared components while ignoring DB positions yields 36 of 39, so the DB-aware Jaccard is always >= the DB-agnostic one. Keep the bound metric default (distances in [0,1]) unless an unbound metric is explicitly requested via Analysis → Unbound distance metric.

## Related tools

- **LipidSpace** (Standalone application that builds lipid molecular graphs, computes the LaWeCSE-based MCS with Jaccard index, and exposes DB-position and sn-position comparison modes) — https://github.com/lifs-tools/lipidspace
- **cppgoslin (Goslin)** (Parses and standardizes lipid shorthand names into structures that feed the graph/MCS comparison) — https://github.com/lifs-tools/cppgoslin
- **lambda-lanczos** (Downstream: computes PCA via the Lanczos method on the n x n Jaccard distance matrix to build the lipid space model) — https://github.com/mrcdr/lambda-lanczos

## Examples

```
shared, united = 38, 39; jaccard_db_aware = shared/united; # FA 18:2(9Z,12Z) vs FA 18:1(9Z): 38/39 with DB positions, 36/39 without
```

## Evaluation signals

- Reference pair FA 18:2(9Z,12Z) vs FA 18:1(9Z): DB-position-aware comparison yields 38 shared of 39 united components; DB-agnostic yields 36 of 39
- Jaccard value lies in [0,1] under the default bound distance metric; comparing a lipid with itself gives distance 0 (similarity 1)
- Intersection count in the DB-aware mode must be >= intersection count in the DB-agnostic mode for the same pair (DB matches never help the agnostic mode)
- Intersection <= union always; both counts drawn only from atom and bond components of the parsed structures
- Written CSV rows match the recorded mode/intersection/union schema for each comparison mode

## Limitations

- By default LipidSpace compares the first FA of one lipid with the first FA of the second, second with second, etc.; comparing all FA combinations (sn-position-agnostic) must be activated via Analysis → Ignore lipid sn-positions, which is more accurate but slower
- In the DB-agnostic mode all double-bond matches count as mismatches, which reduces structural overlap and inflates distances for DB-containing lipids
- LipidSpace analyses cannot currently be saved or loaded, so comparison settings must be re-applied per session
- Enabling the unbound distance metric changes similarity semantics from [0,1] to [0, infinity) and reduces visibility of lipid space models

## Evidence

- [intro] MCS/Jaccard basis of the skill: "we applied the maximum common subgraph
(MCS) approach in combination with the Jaccard index to determine
a pairwise similarity of lipid species based on the molecular structure"
- [methods] LaWeCSE MCS + Jaccard in methods: "we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15"
- [readme] Default FA-chain comparison mode: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid, the second FA of the first lipid with the second FA of the second"
- [readme] Graph-based comparison purpose: "A graph-based comparison of lipid structures allows to calculate distances between lipids and to determine similarities across lipidomes."
