---
name: row-wise-intensity-normalization
description: Use when a MZmine (MZmine2/MZmine3) or GNPS-FBMN feature quantification
  table must be converted to row-wise relative abundances per extract before flagging
  features as specific (min_specificity = 90) for Inventa FC scoring.
when_to_use_negative:
- The feature table is already normalized to relative abundances — re-normalizing
  would distort the ≥90% specificity proportions used by the FC calculation.
- 'The goal is a component that does not depend on feature-intensity normalization:
  the Literature Component (LC) is independent of spectral data, and the Similarity
  Component (SC) uses the MEMO MS2 peak/neutral-loss matrix instead of the aligned
  feature table.'
- Only one intensity column can be used at a time — 'Peak height' and 'Peak area'
  must not be combined in a single normalization pass.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Pipeline that performs the row-wise normalization inside quant_table() and
    consumes it for FC specificity scoring
  repo: https://github.com/luigiquiros/inventa
- name: MZmine3
  role: Source of the quantitative feature table input (also supports MZmine2) that
    is normalized row-wise
  repo: http://mzmine.github.io/
- name: GNPS
  role: Provides the standard FBMN metadata format and GNPS job output consumed alongside
    the normalized table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:239de50e544d439301e0c2c1c4ffe5e95099dc9e3cf0741639d51fff59e9e9f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# row-wise-intensity-normalization

## Summary

Row-wise normalization of an MZmine feature quantification table converts raw peak areas into relative feature abundances per extract, which is the prerequisite step inside Inventa for computing feature specificity and the Feature Component (FC) score.

## When to use

Use when a MZmine (MZmine2/MZmine3) or GNPS-FBMN feature quantification table must be converted to row-wise relative abundances per extract before flagging features as specific (min_specificity = 90) for Inventa FC scoring.

## When NOT to use

- The feature table is already normalized to relative abundances — re-normalizing would distort the ≥90% specificity proportions used by the FC calculation.
- The goal is a component that does not depend on feature-intensity normalization: the Literature Component (LC) is independent of spectral data, and the Similarity Component (SC) uses the MEMO MS2 peak/neutral-loss matrix instead of the aligned feature table.
- Only one intensity column can be used at a time — 'Peak height' and 'Peak area' must not be combined in a single normalization pass.

## Inputs

- MZmine quantitative feature table (.csv, e.g. Celastraceae_pos_quant.csv) with 'Peak area', 'row m/z' and 'row retention time' columns; specify whether Ion Identity was used (data_process_origin='MZMine3', use_ion_identity=True)
- GNPS-format metadata table (.tsv, e.g. Celastraceae_Set_metadata_pos.tsv) with ATTRIBUTE_Species and ATTRIBUTE_Organe headers

## Outputs

- Row-wise normalized feature table (relative feature abundances per extract)
- Output .tsv file containing FC and FS columns for every extract

## How to apply

Load the MZmine quantitative feature table (e.g. Celastraceae_pos_quant.csv, exported with only 'Peak area', 'row m/z' and 'row retention time' columns) and the GNPS-format metadata table (e.g. Celastraceae_Set_metadata_pos.tsv) with pandas inside quant_table() in src/process_data.py. Normalize the intensities row-wise, i.e. divide each extract's (row's) feature intensities by that extract's total intensity to obtain relative feature abundances. A feature is then considered specific within the extract set when at least 90% of its normalized peak area is detected in a single extract (min_specificity = 90). Combine the specificity flag with annotation status — with annotation_preference = 0 only non-annotated features count (ISDB/timaR results filtered at min_score_final = 0.0; SIRIUS annotations at min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0) — and compute FC = (number of specific non-annotated features) / (total features) per extract. Write the FC and FS values for every extract to the output .tsv table.

## Related tools

- **Inventa** (Pipeline that performs the row-wise normalization inside quant_table() and consumes it for FC specificity scoring) — https://github.com/luigiquiros/inventa
- **MZmine3** (Source of the quantitative feature table input (also supports MZmine2) that is normalized row-wise) — http://mzmine.github.io/
- **GNPS** (Provides the standard FBMN metadata format and GNPS job output consumed alongside the normalized table)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('Celastraceae_pos_quant.csv'); df = df.div(df.sum(axis=1), axis=0)"  # row-wise normalization in quant_table() (src/process_data.py)
```

## Evaluation signals

- Each row of the normalized table sums to 1, i.e. every extract's features are expressed as relative abundances.
- A feature is flagged specific only when ≥90% of its normalized peak area is concentrated in one extract (min_specificity = 90).
- The FC and FS values for the Pristimera indica (Willd.) A.C.Sm roots extract reproduce the reported 0.36 and 0.37.
- The output .tsv contains FC and FS columns for every extract in the metadata table.
- With annotation_preference = 0, only non-annotated features (ISDB/timaR at min_score_final = 0.0; SIRIUS at min_ZodiacScore = 0.9, min_ConfidenceScore = 0.0) enter the FC numerator.

## Limitations

- Only one of 'Peak height' or 'Peak area' is considered at a time; using Peak Height requires modifying quant_table() in src/process_data.py.
- Extra exported columns (e.g. Ion Identity annotations) must be removed manually or via df.drop('name of the column', axis=1, inplace=True) inside quant_table(); columns with header 'Unknown: number' are handled by the script.
- The specificity threshold is a user-set parameter (min_specificity = 90 by default); changing it alters which features count as specific and thus the FC value.

## Evidence

- [methods] The Feature Component (FC) is a ratio of the number of specific non-annotated features over the total number of features of each extract.: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [methods] go to src/process_data.py and change it inside the function quant_table(): "go to `src/process_data.py` and change it inside the function quant_table()"
- [readme] if you prefer 'Peak Height', go to src/inventa.py and change it inside the function quand_table().: [evidence span withheld — anchor status: external_doc]
