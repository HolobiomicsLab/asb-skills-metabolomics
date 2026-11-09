---
name: reported-count-reproduction-from-deposited-data
description: Use when a paper reports a database-overlap count and a significance
  test (e.g., 132 of 374 structures in LOTUS, chi-square 32.353) backed by a deposited
  dataset, and you must verify it independently.
when_to_use_negative:
- The target statistic is a model prediction or a result of the multi-omics MEANtools
  workflow itself rather than a database-overlap count reproducible from deposited
  tables.
- The structures are not available as a deposited CSV with per-structure IDs, so per-structure
  matched/unmatched bookkeeping cannot be performed.
- A different metabolite reference database is required instead of LOTUS as downloaded
  on 10/10/2023, changing the reference set that defines the overlap.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: MEANtools
  role: Provides the SQLite-format LOTUS database and the workflow context in which
    the 374 characterized-reaction structures were processed
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: LOTUS
  role: Reference metabolite-structure database queried to determine presence or absence
    of each of the 374 structures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:b1bd1026d4483643c031aa0d100f3dcb99ac8cae89e044b0e496519558f56682
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - LOTUS
  - MEANtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# reported-count-reproduction-from-deposited-data

## Summary

Reproduces a reported statistic from a paper by re-running the matching and significance computation on the deposited source data — here, the 132-of-374 LOTUS structure overlap with chi-square 32.353 (DF=1, p<0.001) — verifying the count and the contingency-table test rather than accepting the reported number.

## When to use

Use when a paper reports a database-overlap count and a significance test (e.g., 132 of 374 structures in LOTUS, chi-square 32.353) backed by a deposited dataset, and you must verify it independently.

## When NOT to use

- The target statistic is a model prediction or a result of the multi-omics MEANtools workflow itself rather than a database-overlap count reproducible from deposited tables.
- The structures are not available as a deposited CSV with per-structure IDs, so per-structure matched/unmatched bookkeeping cannot be performed.
- A different metabolite reference database is required instead of LOTUS as downloaded on 10/10/2023, changing the reference set that defines the overlap.

## Inputs

- Combined_small_middle_big_datasets.csv (374 substrate/product structures from characterized biosynthetic reactions; Zenodo record 15697913)
- LOTUS database in SQLite format (version downloaded 10/10/2023, distributed with MEANtools GitHub repository)

## Outputs

- structure_presence_chisquare.csv containing per-structure match table (matched/unmatched per structure ID) and the chi-square significance result (32.353, DF=1, p<0.001)

## How to apply

Load the 374 substrate/product structures from the deposited characterized-reaction dataset (Combined_small_middle_big_datasets.csv from Zenodo record 15697913). Load the LOTUS database converted to SQLite format, distributed with the MEANtools GitHub repository (version downloaded 10/10/2023). Match each of the 374 structures against the LOTUS structure entries, recording presence (matched) or absence (unmatched) per structure ID. Tabulate the matched/unmatched counts and confirm the reported 132 present out of 374. Then compute a chi-square statistic on the presence/absence contingency with 1 degree of freedom and verify it reproduces 32.353 with p < 0.001. Finally write structure_presence_chisquare.csv containing the per-structure match table and the significance result.

## Related tools

- **MEANtools** (Provides the SQLite-format LOTUS database and the workflow context in which the 374 characterized-reaction structures were processed) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **LOTUS** (Reference metabolite-structure database queried to determine presence or absence of each of the 374 structures)

## Examples

```
python reproduce_lotus_overlap.py --structures Combined_small_middle_big_datasets.csv --lotus lotus.sqlite --output structure_presence_chisquare.csv
```

## Evaluation signals

- Matched-structure count equals exactly 132 out of 374 structures.
- Chi-square statistic on the presence/absence contingency equals 32.353 with DF=1.
- Reported p-value is below 0.001.
- structure_presence_chisquare.csv contains one row per structure ID with a binary matched/unmatched flag plus the significance summary.
- Structure IDs in the output correspond one-to-one with the 374 rows of Combined_small_middle_big_datasets.csv.

## Limitations

- The overlap result is tied to the specific LOTUS version downloaded on 10/10/2023; a newer or older LOTUS snapshot will change the matched count.
- The SQLite-format LOTUS database was specifically converted for MEANtools compatibility, so loading via other tools may need adaptation.
- The chi-square is computed on a 1-degree-of-freedom contingency; any change to how matched/unmatched are tabulated (e.g., counting substrate and product sides separately) invalidates the reported statistic.

## Evidence

- [other] 132 of 374 structures were found in LOTUS with statistical significance (chi-square: 32.353, DF=1, p < 0.001).: [evidence span withheld — anchor status: not_found]
- [discussion] The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository.: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
- [discussion] querying the LOTUS database (downloaded on 10/10/2023): "It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023)."
- [supplementary] can be found at https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv: "can be found at https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv"
