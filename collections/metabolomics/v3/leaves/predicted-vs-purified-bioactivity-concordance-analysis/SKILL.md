---
name: predicted-vs-purified-bioactivity-concordance-analysis
description: Use when a compound isolated via NP Analyst-guided mass-guided fractionation
  must be confirmed as the predominant active component by comparing its measured
  bioassay (BioMAP) profile against the aggregated activity profile of the bioactive
  community's member prefractions.
when_to_use_negative:
- The purified compound's bioassay data exist but no community-level predicted profile
  was ever computed — there is no NP Analyst community to compare against; run the
  compound activity mapping stage first.
- Bioassay columns are not normalized to a common scale — per the article, such data
  are unsuitable for assembly into biological profiles for NP Analyst, so any concordance
  comparison would be invalid.
- The goal is dereplication of known compounds from the MS features alone rather than
  validating an isolation lead against its predicted community activity.
tools:
- name: NP Analyst
  role: CLI/toolchain used to regenerate Activity Scores, Cluster Scores, community
    membership, and the community view of bioactive MS features from the GNPS network
    and normalized bioassay CSVs
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source of the feature network (.graphml) whose community-16 MS features (m/z
    753.3243, fragment 623.2319) define the predicted profile being tested
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:b32c63e535655c1e8c84e68ff93a2e43891613a6b17db12765a8038ba2f688e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - NP Analyst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# predicted-vs-purified-bioactivity-concordance-analysis

## Summary

Validates that the bioactivity profile NP Analyst's Compound Activity Mapping predicts for a bioactive MS-feature community is recapitulated by the measured BioMAP profile of the compound subsequently purified from that community's prefractions — the confirmation step of mass-guided isolation (e.g., amychelin C from community 16).

## When to use

Use when a compound isolated via NP Analyst-guided mass-guided fractionation must be confirmed as the predominant active component by comparing its measured bioassay (BioMAP) profile against the aggregated activity profile of the bioactive community's member prefractions.

## When NOT to use

- The purified compound's bioassay data exist but no community-level predicted profile was ever computed — there is no NP Analyst community to compare against; run the compound activity mapping stage first.
- Bioassay columns are not normalized to a common scale — per the article, such data are unsuitable for assembly into biological profiles for NP Analyst, so any concordance comparison would be invalid.
- The goal is dereplication of known compounds from the MS features alone rather than validating an isolation lead against its predicted community activity.

## Inputs

- GNPS feature network file (.graphml) from the original NP Analyst study
- Normalized bioassay CSVs for the prefraction library (columns on the same scale)
- BioMAP assay profile of the purified/isolated compound (e.g., amychelin C, C31H45N8O14, m/z 753.3053)
- NP Analyst community-view output (retention time vs m/z plot, community membership)

## Outputs

- Community membership table (member prefractions, e.g., 5 for community 16)
- List of connecting bioactive MS features (e.g., 3 features including [M+H]+ 753.3243 and fragment 623.2319)
- Profile-concordance summary comparing predicted community activity with the purified compound's BioMAP profile (e.g., npanalyst_community16_verification.json)

## How to apply

Re-run the NP Analyst CLI pipeline on the deposited data: import the GNPS feature network (.graphml) and normalized bioassay CSVs (`npanalyst import -t GNPS`), then run the replicate/basketing stage (R-tree based basketing of m/z features with overlapping error ranges) and the basketing + compound activity mapping stages with the default configuration (inspect/tune via `npanalyst get_config` → config.json) to compute Activity Scores, Cluster Scores, and bioactive communities. Extract the target community from the community-view output and tabulate its member prefractions and connecting bioactive MS features (community 16 in the amychelin C study: 5 prefractions linked by 3 features, including [M+H]+ 753.3243 and fragment 623.2319). Aggregate the normalized bioassay values of those prefractions into the community's predicted activity profile, ensuring every assay column is on the same scale. Compare that predicted profile against the BioMAP profile measured for the purified compound (amychelin C, C31H45N8O14, m/z 753.3053), checking concordance across the assay panel feature-by-feature. Write the verification report (community membership table, MS feature list, and profile-concordance summary) to a JSON file such as npanalyst_community16_verification.json; full concordance confirms the community's dominant bioactive component was successfully isolated.

## Related tools

- **NP Analyst** (CLI/toolchain used to regenerate Activity Scores, Cluster Scores, community membership, and the community view of bioactive MS features from the GNPS network and normalized bioassay CSVs) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source of the feature network (.graphml) whose community-16 MS features (m/z 753.3243, fragment 623.2319) define the predicted profile being tested)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The reproduced community contains the expected number of member prefractions and connecting bioactive MS features (community 16: 5 prefractions, 3 features).
- The diagnostic MS features are present in the community feature list: [M+H]+ 753.3243 and fragment 623.2319.
- The feature m/z values are consistent with the purified compound's mass (community feature 753.3243 vs amychelin C 753.3053, C31H45N8O14) within the basketing error tolerance.
- The aggregated bioassay profile of the member prefractions recapitulates the reported BioMAP profile of the purified compound across the assay panel.
- The output JSON contains all three required blocks: community membership table, feature list, and profile-concordance summary.

## Limitations

- NP Analyst scores each MS feature independently, so adduct/fragment/multiply-charged features of one molecule form stripes with similar Activity and Cluster Scores but different m/z ratios, requiring manual interpretation when defining the community's feature list.
- Communities are built on bioactive MS feature distribution rather than sample activity profiles, so molecules with different structures but similar biological profiles form separate communities — concordance confirms the community's dominant component, not necessarily every contributing metabolite.
- The workflow was demonstrated with a challenging low-resolution antimicrobial bioassay data set; low-resolution or insensitive assays can inflate network complexity with uninformative low-abundance features if signal intensity cutoffs are set too low.

## Evidence

- [abstract] Compound Activity Mapping as a strategy for predicting compound bioactivity profiles directly from primary screening results on extract libraries: "Compound Activity Mapping as a strategy for predicting compound bioactivity profiles directly from primary screening results on extract libraries"
- [discussion] NP Analyst offers a target-agnostic platform for predicting biological activities of MS features directly from complex mixtures: "NP Analyst offers a target-agnostic platform for predicting biological activities of MS features directly from complex mixtures"
- [discussion] these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
- [methods] docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions).: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
