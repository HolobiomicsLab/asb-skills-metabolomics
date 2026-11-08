---
name: experimental-design-dictionary-construction
description: Use when running PALS programmatically on a peak-intensity matrix (e.g.
  the Rhamnaceae Rhamnus/Ziziphus samples) and sample-factor assignments must be encoded
  as case/control comparisons before building a DataSource.
when_to_use_negative:
- Running PALS from the command line via pals/run_pals.py, where comparisons are supplied
  with the --comparisons flag (e.g. Stage_1/Control) instead of a dictionary
- The dataset has no defined experimental factors or case/control structure (e.g.
  single-condition peak tables), so no comparison can be declared
- Only standard pathway databases (Reactome/KEGG/ChEBI) with the default sample handling
  are needed and no programmatic customisation of groups is required
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PALS
  role: Provides the DataSource constructor and PLAGE/ORA/GSEA methods that consume
    the experimental_design dictionary; comparisons declared here determine the ranked
    pathway output
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Supplies the int_df and annotation_df DataFrames that are passed into DataSource
    alongside the experimental_design dictionary
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:cd258dae4ecbea5283f6817a52e641ff855f5b76e8bc42afad3e34cb40fd8721
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# experimental-design-dictionary-construction

## Summary

Construction of the PALS `experimental_design` dictionary that declares sample-to-group mappings (the 'groups' key) and the case/control comparisons (the 'comparisons' key) needed to instantiate a `DataSource` for programmatic PLAGE/ORA/GSEA pathway ranking.

## When to use

Use when running PALS programmatically on a peak-intensity matrix (e.g. the Rhamnaceae Rhamnus/Ziziphus samples) and sample-factor assignments must be encoded as case/control comparisons before building a DataSource.

## When NOT to use

- Running PALS from the command line via pals/run_pals.py, where comparisons are supplied with the --comparisons flag (e.g. Stage_1/Control) instead of a dictionary
- The dataset has no defined experimental factors or case/control structure (e.g. single-condition peak tables), so no comparison can be declared
- Only standard pathway databases (Reactome/KEGG/ChEBI) with the default sample handling are needed and no programmatic customisation of groups is required

## Inputs

- Peak-intensity pandas DataFrame (rows = peak features, columns = samples) with sample-to-group assignments (e.g. Rhamnus vs Ziziphus factors)
- List of desired comparisons specified as case group, control group, and comparison name
- Peak-to-annotation DataFrame (row_id, entity_id) consumed downstream by the same DataSource

## Outputs

- experimental_design dictionary with 'groups' (factor-to-sample mappings) and 'comparisons' (case/control/name entries)
- A configured pals.feature_extraction.DataSource object ready for PLAGE, ORA, or GSEA decomposition

## How to apply

Parse the 'group' line of the intensity CSV (or equivalent metadata) to map every sample column to its experimental factor, then build a dictionary with two keys: 'groups', which lists all samples belonging to each factor, and 'comparisons', a list of {'case': ..., 'control': ..., 'name': ...} entries such as case Rhamnus vs control Ziziphus for the Rhamnaceae analysis. Pass this dictionary together with `int_df` (intensity matrix, grouping line omitted) and `annotation_df` directly into `pals.feature_extraction.DataSource`. Set `min_replace=5000` so that zero intensities within a declared factor are imputed with the minimum value (or the mean of non-zero samples when only some are zero) — a mis-specified grouping silently corrupts this imputation and the subsequent log-2 transformation and Scipy standardisation to zero mean and unit variance. The comparison names then drive which PLAGE activity scores and p-value/unq_pw_F/tot_ds_F/F_coverage columns are produced by `PLAGE(ds).get_pathway_df()`.

## Related tools

- **PALS** (Provides the DataSource constructor and PLAGE/ORA/GSEA methods that consume the experimental_design dictionary; comparisons declared here determine the ranked pathway output) — https://github.com/glasgowcompbio/PALS
- **pandas** (Supplies the int_df and annotation_df DataFrames that are passed into DataSource alongside the experimental_design dictionary)

## Examples

```
from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, database_name, min_replace=5000)
```

## Evaluation signals

- The dictionary contains a 'comparisons' list in which every entry has 'case', 'control', and 'name' keys, e.g. case Rhamnus / control Ziziphus
- Every sample column of int_df appears in exactly one 'groups' entry, and group names match the labels in the intensity CSV's 'group' line
- The downstream pathway-ranking dataframe contains the expected p-value, unq_pw_F, tot_ds_F and F_coverage columns
- For the Rhamnaceae validation, flavonoid-related motifs (rhamnocitrin, kaempferol, flavonoid core, emodin) rank higher in Rhamnus while the cyclopeptidic-alkaloids motif ranks higher in Ziziphus
- min_replace is explicitly set (default 5000) so zero-intensity imputation within each declared factor behaves as documented

## Limitations

- If all samples in a single experimental factor have zero intensities they are replaced by min_replace (default 5000); incorrectly assigned groups therefore distort imputation, log-2 transformation, and standardisation
- Peaks lacking annotations are excluded from pathway analysis, so the dictionary only affects factors for annotated features
- No software version numbers or changelog are stated in the README, so reproducibility of the dictionary API across releases is unverified

## Evidence

- [readme] The experimental design data in `experimental_design` contains information on groups, which relates all samples in a particular experimental factor: "The experimental design data in `experimental_design` contains information on **groups**, which relates all samples in"
- [readme] 'case': 'beer1', 'control': 'beer2', 'name': 'beer1/beer2': "{'case': 'beer1', 'control': 'beer2', 'name':  'beer1/beer2'}"
- [methods] When PALS is used programatically, pandas dataframes storing relevant data can be passed directly.: "When PALS is used programatically, pandas dataframes storing relevant data can be passed directly."
- [readme] the second line can be used to indicate which groups this sample belongs to: "If using the CSV file, the second line can be used to indicate which groups this sample belongs to."
