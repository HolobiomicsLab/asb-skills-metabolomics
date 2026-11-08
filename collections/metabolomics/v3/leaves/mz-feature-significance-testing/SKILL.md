---
name: mz-feature-significance-testing
description: Use when a raw untargeted metabolomics feature table of thousands of
  spectral features (e.g. 5,707 or 7,995 peaks) must be reduced to a significant list
  for downstream pathway/network analysis, and feature selection is the upstream step.
when_to_use_negative:
- Input is already a curated annotation table with identified metabolites — significance
  testing on features is unnecessary and mummichog can consume annotations directly
- The analysis goal is targeted metabolite quantification rather than untargeted pathway/network
  prediction
- Feature table has not been through peak picking and alignment (raw mzXML/mzML spectra),
  since a coherent feature table with per-sample intensities is a prerequisite
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MAVEN
  role: Performs untargeted feature extraction, peak picking, quantification and alignment
    from deposited mzXML files to produce the feature table that feeds significance
    testing
- name: mummichog
  role: Downstream consumer of the significant list and reference list produced by
    this skill; it computes possibly matched metabolites, pathway enrichment and network
    modules
  repo: http://atcg.googlecode.com
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:aa20e524dba30f345cc8cd11f326858be32c03098f656c13ef24e7d20b331fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - MAVEN
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mz-feature-significance-testing

## Summary

Selects statistically significant m/z spectral features from an untargeted LC/MS feature table (e.g. by univariate t-test) to serve as the 'significant list' input that mummichog uses for pathway and network module prediction without metabolite identification.

## When to use

Use when a raw untargeted metabolomics feature table of thousands of spectral features (e.g. 5,707 or 7,995 peaks) must be reduced to a significant list for downstream pathway/network analysis, and feature selection is the upstream step.

## When NOT to use

- Input is already a curated annotation table with identified metabolites — significance testing on features is unnecessary and mummichog can consume annotations directly
- The analysis goal is targeted metabolite quantification rather than untargeted pathway/network prediction
- Feature table has not been through peak picking and alignment (raw mzXML/mzML spectra), since a coherent feature table with per-sample intensities is a prerequisite

## Inputs

- Aligned and quantified LC/MS feature table (rows = spectral features with m/z, retention time, per-sample intensities)
- Sample-group labels defining the biological contrast (e.g. mutant vs wild-type, infected vs control)

## Outputs

- Significant list of m/z features with p-values (e.g. 426 features for yeast, 601 for moDC)
- Reference list of all features detected in the MS experiment (e.g. 5,707 or 7,995 features)

## How to apply

Start from the aligned, quantified feature table produced after peak extraction and alignment of LC/MS data, containing columns for m/z, retention time and intensity per sample. For a two-group contrast (e.g. mutant vs prototrophic wild-type yeast, or infected vs control moDC), apply a univariate statistical test such as the t-test per feature to obtain a p-value, then select the significant list by a cutoff value. In the moDC dataset 601 of 7,995 features were significant, and in the yeast dataset 426 of 5,707 features were selected; both lists plus the full reference list are then passed to mummichog. The rationale is that collective power in metabolic networks resolves ambiguity in metabolite prediction, so a statistically filtered subset preserves the signal needed for pathway enrichment (Fisher's exact test with EASE correction) and module scoring.

## Related tools

- **MAVEN** (Performs untargeted feature extraction, peak picking, quantification and alignment from deposited mzXML files to produce the feature table that feeds significance testing)
- **mummichog** (Downstream consumer of the significant list and reference list produced by this skill; it computes possibly matched metabolites, pathway enrichment and network modules) — http://atcg.googlecode.com

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- The significant list size falls in the expected order of magnitude relative to the reference list — e.g. 601 of 7,995 (moDC) or 426 of 5,707 (yeast) features selected
- Each retained feature carries both an m/z value and a p-value from the statistical test so it can be matched by mummichog
- A cutoff value is explicitly documented for the chosen contrast rather than an ad-hoc filter
- The reference list equals the total number of spectral features detected after QC, matching the count reported by the upstream alignment step
- Downstream mummichog run produces pathway enrichment (Fisher's exact test with EASE correction) and network module results, indicating the significant list was consumable

## Limitations

- Quantification and feature selection, the steps upstream of mummichog, can still be problematic
- Sometimes a single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Current mummichog version does not incorporate a statistical matrix from the feature selection step prior to its analysis
- Features from a high-resolution profiling experiment by far exceed current annotations in metabolite databases, leading to a de facto filtering during mummichog analysis

## Evidence

- [results] mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment).: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [discussion] In the moDC data, we chose by a cutoff value.: "In the moDC data, we chose  by a cutoff value ."
- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were significantly different between the infected samples: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were"
- [readme] User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID.: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
- [discussion] Quantification and feature selection, the steps upstream of mummichog, can still be problematic.: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
