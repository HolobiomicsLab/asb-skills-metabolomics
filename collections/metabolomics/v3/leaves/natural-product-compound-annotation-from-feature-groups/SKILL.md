---
name: natural-product-compound-annotation-from-feature-groups
description: Use when a NP Analyst community lacks a single dominant bioactive MS
  feature but contains multiple high Activity-Score features that may represent adducts/fragments
  of the same molecule; group them by retention time to dereplicate constituent compounds.
when_to_use_negative:
- The community already has a single clear individual bioactive MS feature that can
  be dereplicated directly without stripe grouping
- Input is raw mass-list data rather than processed MZmine 2 or GNPS feature output
  — preprocessing is required first to remove junk features
- The community's features do not form coherent retention-time stripes (no shared
  RT or Activity/Cluster Score pattern), so stripe-based grouping would be uninformative
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NP Analyst
  role: Produces the scored MS features and community views (retention time vs m/z
    plot) from which feature stripes are identified and dereplicated
  repo: https://github.com/liningtonlab/npanalyst
- name: pandas
  role: Loads the community feature table, groups features by retention-time cluster,
    and writes the dereplication assignment CSV
- name: MZmine 2
  role: Generates the processed MS feature peak lists that serve as NP Analyst input
- name: GNPS
  role: Alternative source of processed/networked MS feature files for NP Analyst
    input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:3cdfe43b3a85525d074d29b0be936a0d7b2191d919d81d029fa0ba3bbf7523c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# natural-product-compound-annotation-from-feature-groups

## Summary

Annotates bioactive molecules within an NP Analyst community by grouping scored MS features into retention-time stripes of adducts, fragments, and isotopes, then dereplicating each stripe to a compound (e.g., collismycin A, SF2738D, collismycin B isobar in community 15).

## When to use

Use when a NP Analyst community lacks a single dominant bioactive MS feature but contains multiple high Activity-Score features that may represent adducts/fragments of the same molecule; group them by retention time to dereplicate constituent compounds.

## When NOT to use

- The community already has a single clear individual bioactive MS feature that can be dereplicated directly without stripe grouping
- Input is raw mass-list data rather than processed MZmine 2 or GNPS feature output — preprocessing is required first to remove junk features
- The community's features do not form coherent retention-time stripes (no shared RT or Activity/Cluster Score pattern), so stripe-based grouping would be uninformative

## Inputs

- NP Analyst community feature table (m/z, retention time, Activity Score, Cluster Score per feature)
- Deposited NP Analyst MS data from MZmine 2 or GNPS processed peak lists
- Extracted-ion chromatograms (EICs) for peak-shape confirmation

## Outputs

- Feature-to-compound assignment table (e.g., community15_dereplication.csv) mapping each MS feature to a compound, retention-time set, and adduct/fragment/isotope rationale

## How to apply

Load the community's feature table containing the strong-Activity-Score MS features with m/z, retention time, Activity Score, and Cluster Score. Because NP Analyst scores each MS feature independently, all features from one bioactive molecule carry similar Activity and Cluster Scores but different m/z ratios, so group features by retention-time clusters (in community 15: 2.64, 3.42, and 3.97 min). Within each RT set, compare m/z differences and offsets to classify features as adducts, fragments, or isotopes of a single parent; confirm co-retention with extracted-ion chromatogram (EIC) peak-shape grouping. Assign each stripe to a compound by dereplication — in community 15 the 2.64 min set is collismycin A, 3.42 min is SF2738D, and 3.97 min is a collismycin B isobar. Write the feature-to-compound assignments, retention times, and rationale (e.g., community15_dereplication.csv) using pandas. Feature lists should come from preprocessed MZmine 2 or GNPS output rather than raw mass lists.

## Related tools

- **NP Analyst** (Produces the scored MS features and community views (retention time vs m/z plot) from which feature stripes are identified and dereplicated) — https://github.com/liningtonlab/npanalyst
- **pandas** (Loads the community feature table, groups features by retention-time cluster, and writes the dereplication assignment CSV)
- **MZmine 2** (Generates the processed MS feature peak lists that serve as NP Analyst input)
- **GNPS** (Alternative source of processed/networked MS feature files for NP Analyst input)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The output CSV reproduces three retention-time sets (2.64, 3.42, 3.97 min) that together cover all six strong-Activity-Score features of community 15
- Each RT set is assigned to the expected compound: 2.64 min → collismycin A, 3.42 min → SF2738D, 3.97 min → collismycin B isobar
- Within each group, features share similar Activity and Cluster Scores but distinct m/z values, consistent with the NP Analyst independent-scoring convention
- m/z offsets within a group are explainable as adducts, fragments, or isotopes of a single parent molecule
- EIC peak shapes of co-grouped features co-elute, supporting the RT-based grouping

## Limitations

- NP Analyst scores each MS feature independently, so stripe interpretation (adducts/fragments/multiply-charged species) requires manual judgment
- Some communities lack clear individual bioactive MS features, and the RT-stripe strategy may still fail if features do not share retention times
- Replicate comparison is only possible when replicate data are available; without preprocessing, junk features inflate the candidate pool and complicate grouping
- Low minimum intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features

## Evidence

- [discussion] Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [discussion] identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view: "identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions).: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
