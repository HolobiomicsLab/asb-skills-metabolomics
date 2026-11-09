---
name: inchikey-block-comparison
description: Use when a candidate list retrieved from PubChem, ChemSpider, KEGG, or
  a user-supplied SDF contains stereoisomeric duplicates (shared first InChIKey block)
  that would otherwise inflate candidate counts and split score mass across chemically
  equivalent entries.
when_to_use_negative:
- Stereochemistry itself is the discriminative signal of interest — the first InChIKey
  block is identical across stereoisomers, so this filter deliberately discards all
  but one stereoisomer per skeleton
- The candidate list is already dereplicated (e.g., converted from a suspect list
  of unique InChIKeys, as done for STOFF-IDENT in MetFrag2.2), so no first-block redundancy
  exists to remove
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: MetFrag2.2
  role: Implements the InChIKey filtering step that keeps only the best-scored candidate
    per first InChIKey block to reduce candidate redundancy due to stereoisomerism
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragRelaunched
  role: Open-source Java codebase (MetFragLib, MetFragCommandLine) where the candidate
    filtering pipeline is built and run
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: PubChem
  role: Source of candidate structures whose stereoisomeric redundancy motivates the
    first-block InChIKey grouping
- name: ChemSpider
  role: Alternative compound database from which candidate structures are retrieved
    prior to InChIKey filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:21e308cedfe7096a1e8ebe999e325fa50daefa0c899268d6f67bbb32b8379799
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpider
  - MetFrag2.2
  - MetFragRelaunched
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# InChIKey block comparison

## Summary

A candidate pre-filtering technique from MetFrag2.2 that groups candidate structures by the first InChIKey block (molecular skeleton, which excludes stereochemistry) and retains only the single best-scored representative per group, removing redundant stereoisomers before further scoring or fragmentation.

## When to use

Use when a candidate list retrieved from PubChem, ChemSpider, KEGG, or a user-supplied SDF contains stereoisomeric duplicates (shared first InChIKey block) that would otherwise inflate candidate counts and split score mass across chemically equivalent entries.

## When NOT to use

- Stereochemistry itself is the discriminative signal of interest — the first InChIKey block is identical across stereoisomers, so this filter deliberately discards all but one stereoisomer per skeleton
- The candidate list is already dereplicated (e.g., converted from a suspect list of unique InChIKeys, as done for STOFF-IDENT in MetFrag2.2), so no first-block redundancy exists to remove

## Inputs

- Candidate structure list from PubChem, ChemSpider, or KEGG database retrieval (per-structure identifiers, SMILES/InChI, formulas, scores)
- User-supplied structure data file (SDF)
- Per-candidate MetFrag2.2 in silico fragmentation scores

## Outputs

- Reduced candidate list with at most one representative per first InChIKey block (best-scored stereoisomer retained)
- Mapping of discarded stereoisomers to their retained skeleton representative (for provenance)

## How to apply

Compute or obtain the InChIKey for every candidate in the list (candidates may come from PubChem/ChemSpider retrieval or a user-supplied SDF), then group candidates that share the first InChIKey block, which encodes the molecular skeleton independent of stereochemistry. Run the candidate scoring (e.g., in silico fragmentation score from MetFrag2.2) and, within each first-block group, keep only the best-scored candidate, discarding the remaining stereoisomers. In the MetFrag2.2 results workflow this filtering step — 'Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates' InChIKeys' — is applied after initial scoring so that stereochemistry-driven redundancy is removed without losing the top-scoring structure. The reduced candidate set then proceeds to downstream filters/scoring such as element restrictions (include/require/exclude modes) and SMARTS substructure matching implemented via the Chemistry Development Kit. Rationale: databases like PubChem have roughly doubled since 2009, so collapsing stereoisomeric duplicates makes ranking less challenging and concentrates evaluation on distinct skeletons.

## Related tools

- **MetFrag2.2** (Implements the InChIKey filtering step that keeps only the best-scored candidate per first InChIKey block to reduce candidate redundancy due to stereoisomerism) — http://c-ruttkies.github.io/MetFrag/
- **MetFragRelaunched** (Open-source Java codebase (MetFragLib, MetFragCommandLine) where the candidate filtering pipeline is built and run) — https://github.com/ipb-halle/MetFragRelaunched
- **PubChem** (Source of candidate structures whose stereoisomeric redundancy motivates the first-block InChIKey grouping)
- **ChemSpider** (Alternative compound database from which candidate structures are retrieved prior to InChIKey filtering)

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git && mvn clean install -pl MetFragCommandLine -am
```

## Evaluation signals

- After filtering, no two retained candidates share the same first InChIKey block (one representative per skeleton group)
- The retained representative in each group is the highest-scoring candidate of that group under the fragmentation score
- The number of candidates drops from the retrieved count to the number of distinct first InChIKey blocks, quantifying stereoisomer redundancy removed
- Downstream element-filter and SMARTS substructure scoring (W_include = W_exclude = 0.5, Eq. 3-4 normalization) operate on the post-filter list without re-introducing duplicates

## Limitations

- The filter collapses all stereoisomers sharing a first InChIKey block to one candidate, so the correct stereoisomer may be discarded if a higher-scoring isomer of the same skeleton exists
- Applicable only when per-candidate scores exist to select the best representative; an unfiltered score must be computed before the group-wise selection
- MetFrag2.2 project metadata gaps: no version number stated beyond the project name and no changelog found in the source repository

## Evidence

- [abstract] InChIKey filtering to reduce candidate redundancy due to stereoisomerism: "as well as InChIKey filtering to reduce candidate redundancy due to stereoisomerism"
- [results] First-block comparison semantics: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [intro] Motivation: growing databases make ranking harder: "As PubChem is now double the size of the 2009 version, the candidate ranking becomes more challenging over time"
- [intro] Candidate retrieval sources feeding the filter: [evidence span withheld — anchor status: not_found]
- [readme] Build/run entry point for the library implementing the filter: "mvn clean install -pl MetFragLib -am"
