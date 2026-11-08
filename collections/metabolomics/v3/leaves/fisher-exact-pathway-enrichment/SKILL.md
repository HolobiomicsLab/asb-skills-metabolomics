---
name: fisher-exact-pathway-enrichment
description: Use when an untargeted LC/MS feature table (m/z, retention time, p-value)
  must be mapped onto pathway-level biological activity, and metabolite identities
  are only tentatively matched, so enrichment must tolerate many-to-many m/z-to-metabolite
  matches.
when_to_use_negative:
- 'The goal is confident metabolite identification: ''Mummichog is not designed to
  replace tandem mass spectrometry in metabolite identification'', so confirmed annotation
  (MS/MS or authentic standards) should not be claimed from this enrichment step.'
- The input is already a list of unambiguously identified metabolites, where standard
  over-representation testing on identified compounds avoids the tentative-matching
  ambiguity this skill must correct for.
- 'Quantification and feature selection upstream are unreliable: ''Quantification
  and feature selection, the steps upstream of mummichog, can still be problematic'',
  and garbage-in feature tables will invalidate the enrichment statistics.'
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Core package that computes possibly matched metabolites from m/z features,
    runs Fisher's exact test per pathway, fits the permutation null p-value distribution
    as Gamma, and emits pathway result tables, result.html and JSON strings.
  repo: https://github.com/shuzhao-li/mummichog
- name: metDataModel
  role: Structures the pathway/reaction/compound definitions (list_of_pathways, list_of_reactions,
    list_of_compounds) and feature annotations that supply the contingency-table membership
    used in the Fisher's exact test.
- name: cloud-mummichog
  role: RESTful server/worker implementation for running the mummichog pathway analysis
    pipeline remotely.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:2de41cd511cc00817c7912597b5b6e9ceaabd992c9afddf0955d4c0617a76270
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/mummichog
  tools:
  - Azimuth DB
  - cloud-mummichog
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# fisher-exact-pathway-enrichment

## Summary

Pathway-level enrichment testing in mummichog that runs Fisher's exact test on tentatively matched metabolites against a genome-scale metabolic model, with a permutation-based null p-value distribution modeled as Gamma and EASE-style smaller-of correction to control overlap inflation.

## When to use

Use when an untargeted LC/MS feature table (m/z, retention time, p-value) must be mapped onto pathway-level biological activity, and metabolite identities are only tentatively matched, so enrichment must tolerate many-to-many m/z-to-metabolite matches.

## When NOT to use

- The goal is confident metabolite identification: 'Mummichog is not designed to replace tandem mass spectrometry in metabolite identification', so confirmed annotation (MS/MS or authentic standards) should not be claimed from this enrichment step.
- The input is already a list of unambiguously identified metabolites, where standard over-representation testing on identified compounds avoids the tentative-matching ambiguity this skill must correct for.
- Quantification and feature selection upstream are unreliable: 'Quantification and feature selection, the steps upstream of mummichog, can still be problematic', and garbage-in feature tables will invalidate the enrichment statistics.

## Inputs

- Feature table with m/z, retention time, and p-value from a univariate statistical test (e.g., TSV such as tests/ineurons_ttest_1127.tsv)
- Significant feature list (e.g., selected by cutoff) and reference list of all detected features
- Metabolic model as list_of_pathways, list_of_reactions, list_of_compounds, with correct neutral-mass chemical formulas (e.g., KEGG/Recon1/Edinburgh human network; BioCyc yeast model)
- Optional annotation table on features (e.g., empCpds_with_annotations.json from authentic standards or MS/MS)

## Outputs

- Pathway enrichment table with Fisher's exact test p-values and permutation/Gamma-derived adjusted p-values per pathway
- Result tables and figures in result.html
- JSON strings from pathway analysis (and network module analysis) for programmatic use

## How to apply

Load the user-supplied feature table (m/z, rtime, p-value) together with a metabolic model formatted as list_of_pathways, list_of_reactions and list_of_compounds, requiring correct neutral (non-salt) formulas so adducts can be computed. For each pathway, tabulate the overlap between possibly matched metabolites from the significant feature list and pathway members, and run Fisher's exact test against the reference network background. Because a single m/z feature can match several metabolites in the same module/pathway and inflate the overlap number, apply the smaller-of (EASE score) correction to the overlap count. Build the null p-value distribution by random permutation of m/z features and model it as a Gamma distribution to derive a per-pathway adjusted p-value. Report pathway enrichment in result tables, figures (result.html) and JSON strings; the moDC/YF17D validation case (7,995 features, 601 significant) is the reference result against which outputs should be checked.

## Related tools

- **mummichog** (Core package that computes possibly matched metabolites from m/z features, runs Fisher's exact test per pathway, fits the permutation null p-value distribution as Gamma, and emits pathway result tables, result.html and JSON strings.) — https://github.com/shuzhao-li/mummichog
- **metDataModel** (Structures the pathway/reaction/compound definitions (list_of_pathways, list_of_reactions, list_of_compounds) and feature annotations that supply the contingency-table membership used in the Fisher's exact test.)
- **cloud-mummichog** (RESTful server/worker implementation for running the mummichog pathway analysis pipeline remotely.)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Verify each pathway p-value derives from a Fisher's exact test whose overlap count was reduced by the smaller-of (EASE) correction, not the raw possibly-matched overlap.
- Confirm a permutation-based null p-value distribution was generated by random permutation of m/z features and modeled as a Gamma distribution before adjusted p-values were computed.
- Check that inputs to the test came from tentative matching with confidence rating (e.g., co-presence of M+H[1+] and M(C13)+H[1+] forms as high confidence), since matched-metabolite counts feed the contingency table.
- For the moDC/YF17D activation dataset, significant pathways in result.html should be consistent with the published moDC analysis.
- Confirm the model used chemical formulas as neutral mass (not salt format), since wrong formulas corrupt adduct-based matching and thus the enrichment counts.

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number and biasing the Fisher's exact test toward significance; the smaller-of correction only partially mitigates this.
- De facto filtering occurs because features from high-resolution profiling experiments far exceed current annotations in metabolite databases, so pathways are tested only over annotated compounds.
- A number of metabolites in metabolomics databases are not present in any of the metabolic models, and models need improvement on lipid annotation, physiological context and tissue specificity.
- Quantification and feature selection, the steps upstream of mummichog, can still be problematic, and the current version does not incorporate a statistical matrix from the feature selection step or mass flow balance of metabolic reactions.

## Evidence

- [results] FET is the core test: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [discussion] EASE/smaller-of correction rationale: "Sometimes, a single m/z feature may match to several metabolites in the same module/pathway, inflating the overlap number."
- [readme] Output formats: "JSON strings from pathway analysis and network module analysis for programmatic use."
- [readme] Metabolic model format requirement: "The format of a metabolic model contains list_of_pathways, list_of_reactions and list_of_compounds."
- [results] Reference model basis: "The genome-scale human metabolic network in mummichog is based on KEGG [38], UCSD Recon1 [15] and Edinburgh human metabolic network [16]."
