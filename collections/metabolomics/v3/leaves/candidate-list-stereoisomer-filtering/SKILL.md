---
name: candidate-list-stereoisomer-filtering
description: Use when a candidate structure list (from PubChem, ChemSpider, KEGG,
  or a user SDF) contains stereoisomeric duplicates that would crowd out distinct
  scaffolds in the ranked MetFrag output.
when_to_use_negative:
- When stereoisomeric identity itself is the question (e.g. chiral LC separations),
  since the filter intentionally discards all but one stereoisomer per skeleton.
- When the candidate list comes from a stereochemistry-unaware source that has already
  collapsed InChIKey blocks, as grouping would be a no-op.
- When ranking must be preserved for all candidates (e.g. suspect-list confirmation),
  because removing stereoisomers changes the downstream rank distribution.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: MetFrag2.2
  role: Implements the first-InChIKey-block stereoisomer filter as part of its candidate
    post-processing
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFragCommandLine
  role: CLI entry point (metfragcl) that applies the filter during a MetFrag run;
    built via Maven from MetFragCommandLine module
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:1251063df496c1404b61ef211b66e6186bd6ce7ceba26b6348f337bfe7c3f2c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - MetFrag2.2
  - MetFragCommandLine
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Candidate list stereoisomer filtering (InChIKey first-block)

## Summary

Reduces redundant candidate structures by grouping compounds sharing the first InChIKey block (identical molecular skeleton, differing only in stereochemistry) and retaining the single best-scored representative per group, as implemented in MetFrag2.2. This shrinks candidate lists before/after scoring so stereoisomers do not occupy multiple high ranks.

## When to use

Use when a candidate structure list (from PubChem, ChemSpider, KEGG, or a user SDF) contains stereoisomeric duplicates that would crowd out distinct scaffolds in the ranked MetFrag output.

## When NOT to use

- When stereoisomeric identity itself is the question (e.g. chiral LC separations), since the filter intentionally discards all but one stereoisomer per skeleton.
- When the candidate list comes from a stereochemistry-unaware source that has already collapsed InChIKey blocks, as grouping would be a no-op.
- When ranking must be preserved for all candidates (e.g. suspect-list confirmation), because removing stereoisomers changes the downstream rank distribution.

## Inputs

- Ranked candidate structure list from MetFrag (with InChIKey or SMILES/InChI per candidate)
- InChIKey strings per candidate (first block used for grouping)
- MetFrag score values per candidate (fragment score plus optional reference/retention/user scores)

## Outputs

- Deduplicated candidate list with one representative per first-InChIKey-block group
- Report noting how many stereoisomeric duplicates were removed

## How to apply

After MetFrag scoring, compute each candidate's InChIKey and group candidates by the first block, which encodes the molecular skeleton but not stereochemistry. Keep only the single best-scored candidate within each first-block group and drop the remaining stereoisomers; the paper's EQExPlus dataset applied exactly this rule, retaining one representative per skeleton. This is a post-scoring redundancy filter, not a mass or formula filter, so it is applied after the fragment-score (and optional reference/retention) weights have been computed. In MetFragCL the behavior is controlled via the LocalPubChemDatabaseInChIKey1Column (or InChIKey-based candidate fields), and it can be combined with element restrictions (e.g. included-elements = C, S) and SMARTS substructure scoring. Verify the filter by confirming that no two surviving candidates share the same first InChIKey block and that the top-ranked candidate's score was maximal within its group.

## Related tools

- **MetFrag2.2** (Implements the first-InChIKey-block stereoisomer filter as part of its candidate post-processing) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFragCommandLine** (CLI entry point (metfragcl) that applies the filter during a MetFrag run; built via Maven from MetFragCommandLine module) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
metfragcl -c candidate.sdf -m precursors.txt -t /path/to/CompoundName --filter-inchikey-first-block true
```

## Evaluation signals

- No two remaining candidates share the same first InChIKey block.
- For each retained candidate, its total MetFrag score is greater than or equal to the scores of all dropped stereoisomers in the same group.
- Filter does not change the relative order of candidates belonging to different first-block groups.
- Applied only when InChIKey (or equivalent) is available; candidates missing an InChIKey should be flagged rather than silently grouped.

## Limitations

- Keeps only one stereoisomer per skeleton, so it cannot resolve which stereoisomer is correct when no further evidence (e.g. chiral retention) exists.
- Requires that a total score already exists — applied after fragment scoring, so an unranked list cannot be deduplicated this way.
- The paper notes that as databases like PubChem grow, the candidate ranking becomes more challenging; this filter mitigates but does not eliminate redundancy.

## Evidence

- [abstract] inChIKey filtering to reduce candidate redundancy due to stereoisomerism: "as well as InChIKey filtering to reduce candidate redundancy due to stereoisomerism"
- [results] Stereoisomer filter implementation: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [intro] Database growth motivation: "As PubChem is now double the size of the 2009 version, the candidate ranking becomes more challenging over time"
- [readme] MetFrag source availability: "download sources by cloning git repository<br>
```bash
git clone https://github.com/ipb-halle/MetFragRelaunched.git"
