---
name: ms-feature-adduct-fragment-interpretaion
description: Use when NP Analyst (or equivalent Compound Activity Mapping) output
  shows several distinct m/z features with similar Activity and Cluster Scores sharing
  one retention time, indicating adduct/fragment/multiply-charged stripes of one molecule.
when_to_use_negative:
- Input is already a basketed/replicate-merged feature table where adducts and fragments
  have been collapsed into single entries
- The goal is computing Activity Scores or Cluster Scores themselves — NP Analyst
  performs that computation; this skill only interprets its per-feature output
- Bioactivity scores are absent (e.g., raw unprocessed mass lists with no bioassay
  association), so stripes cannot be distinguished from noise features
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: NP Analyst
  role: Source of the per-feature Activity Scores, Cluster Scores, and community-view
    retention time vs m/z plot that this interpretation step operates on
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Provides the .graphml feature network files imported into NP Analyst whose
    MS features (adducts/fragments) are then grouped by retention-time stripe
  repo: https://github.com/liningtonlab/npanalyst
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:1926ab63a388eeeb8f36b259a98316df9079f12a43a2babda99183c1e6073b24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - NP Analyst
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# MS feature adduct/fragment interpretation

## Summary

Manual interpretation step that groups independently scored NP Analyst MS features belonging to the same metabolite (adducts, in-source fragments, multiply-charged species) by co-located retention time and shared Activity/Cluster Scores, so a single bioactive molecule (e.g., [M+H]+ 753.3243 and its 623.2319 fragment for amychelin C) is treated as one prioritization target rather than separate features.

## When to use

Use when NP Analyst (or equivalent Compound Activity Mapping) output shows several distinct m/z features with similar Activity and Cluster Scores sharing one retention time, indicating adduct/fragment/multiply-charged stripes of one molecule.

## When NOT to use

- Input is already a basketed/replicate-merged feature table where adducts and fragments have been collapsed into single entries
- The goal is computing Activity Scores or Cluster Scores themselves — NP Analyst performs that computation; this skill only interprets its per-feature output
- Bioactivity scores are absent (e.g., raw unprocessed mass lists with no bioassay association), so stripes cannot be distinguished from noise features

## Inputs

- NP Analyst community-view output (retention time vs m/z plot, community membership table with member prefractions and connecting bioactive MS features)
- NP Analyst scatter-plot and network views with per-feature Activity Scores and Cluster Scores
- GNPS feature network .graphml import or MZmine 2 peak-list-derived feature table feeding NP Analyst

## Outputs

- De-duplicated list of candidate bioactive metabolites, each defined by an adduct/fragment stripe (e.g., [M+H]+ m/z plus co-retained fragment m/z values)
- Aggregated bioassay activity profile assigned to each metabolite stripe across its member prefractions
- Priority-metabolite annotation used to guide mass-guided isolation and later confirmation against purified compound data

## How to apply

Because NP Analyst scores each MS feature independently, every adduct, in-source fragment, and multiply-charged form of one bioactive metabolite appears as its own row with different m/z but near-identical Activity and Cluster Scores. Inspect the community view plot of retention time vs m/z ratio to identify vertical stripes of active features at the same retention time, then cross-check the scatter and network views — the three viewpoints are meant to be used interdependently to confirm the grouping. Aggregate the associated bioassay profile across the stripe's member prefractions to judge the molecule's activity, not each feature in isolation. Use the stripe pattern to de-duplicate candidate metabolites before isolation (e.g., recognizing that community-16's m/z 753.3243 [M+H]+ and fragment 623.2319 describe one compound) and to interpret purified-compound data afterwards, as done when purified amychelin C (C31H45N8O14, m/z 753.3053) was matched back to the community's predicted profile.

## Related tools

- **NP Analyst** (Source of the per-feature Activity Scores, Cluster Scores, and community-view retention time vs m/z plot that this interpretation step operates on) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Provides the .graphml feature network files imported into NP Analyst whose MS features (adducts/fragments) are then grouped by retention-time stripe) — https://github.com/liningtonlab/npanalyst

## Evaluation signals

- All features assigned to one metabolite stripe share the same retention time and show similar (not identical) Activity and Cluster Scores but distinct m/z ratios
- Stripe membership is consistent across the community, scatter, and network views rather than appearing as an artifact of a single visualization
- The aggregated bioassay profile of the prefractions containing the stripe recapitulates the BioMAP profile measured for the subsequently isolated compound (as verified for amychelin C)
- Molecular formula/mass of the isolated compound (e.g., m/z 753.3053 for C31H45N8O14) is consistent with the observed [M+H]+ and fragment m/z values in the stripe
- Number of distinct candidate metabolites equals the number of retention-time stripes, not the raw count of bioactive MS features

## Limitations

- NP Analyst scores each MS feature independently, so adduct/fragment grouping is a manual interpretive step and different adducts of one molecule can appear as separate candidates if the stripe is missed
- Some bioactive communities lack clear individual bioactive MS features, in which case stripe-based prioritization cannot be applied and retention-time-set inspection is the fallback
- Replicate comparison and appropriate signal intensity cutoffs are recommended upstream; without them noise peaks can masquerade as co-retained features and inflate stripe counts

## Evidence

- [discussion] NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule share similar scores: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [discussion] Retention-time stripe strategy for communities without clear individual bioactive MS features: "In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same"
- [discussion] The scatter, network, and community views should be used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [results] Task card: community 16 connects prefractions via the 753.3243 [M+H]+ and 623.2319 fragment, later confirmed as amychelin C: [evidence span withheld — anchor status: not_found]
- [readme] README: R-tree basketing combines overlapping m/z features, the replicate-merging step preceding feature interpretation: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged)"
