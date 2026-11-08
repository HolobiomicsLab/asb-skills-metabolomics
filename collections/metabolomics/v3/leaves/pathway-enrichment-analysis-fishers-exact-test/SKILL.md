---
name: pathway-enrichment-analysis-fishers-exact-test
description: Use when an untargeted LC/MS metabolomics run yields two m/z feature
  lists — a significant list and a reference list — and you need pathway-level biological
  activity without prior metabolite identification.
when_to_use_negative:
- A metabolite-identity table is already available and standard over-representation
  tools (GO/KEGG ORA) suffice — this skill is specifically for feature-level (pre-identification)
  enrichment.
- The goal is to identify individual metabolites rather than pathway-level activity;
  mummichog is explicitly not a replacement for tandem mass spectrometry.
- The input is a targeted panel of pre-identified metabolites where feature-level
  ambiguity is absent.
- Data are from a single feature rather than a feature table (no statistical feature
  selection was performed).
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: mummichog
  role: Core package that computes possibly matched metabolites from m/z, runs Fisher's
    exact test with EASE score per pathway, and fits a Gamma distribution to the permutation
    p-value null to assign adjusted p-values.
  repo: http://atcg.googlecode.com
- name: cloud-mummichog
  role: RESTful server/worker implementation of the same pathway enrichment pipeline,
    usable when the analysis is served from a web front-end.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:5aabd38842c87aac5bb0589cc2347aa0778917202ab9726e9c07836541afd5b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cloud-mummichog
  - mummichog
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pathway-enrichment-analysis-fishers-exact-test

## Summary

Pathway-level enrichment testing of LC/MS spectral features against genome-scale metabolic models using Fisher's exact test with an EASE-score correction and a Gamma-modeled permutation null for adjusted p-values, as implemented in mummichog for untargeted metabolomics.

## When to use

Use when an untargeted LC/MS metabolomics run yields two m/z feature lists — a significant list and a reference list — and you need pathway-level biological activity without prior metabolite identification.

## When NOT to use

- A metabolite-identity table is already available and standard over-representation tools (GO/KEGG ORA) suffice — this skill is specifically for feature-level (pre-identification) enrichment.
- The goal is to identify individual metabolites rather than pathway-level activity; mummichog is explicitly not a replacement for tandem mass spectrometry.
- The input is a targeted panel of pre-identified metabolites where feature-level ambiguity is absent.
- Data are from a single feature rather than a feature table (no statistical feature selection was performed).

## Inputs

- significant m/z feature list (e.g. 601 features at p cutoff in the moDC YF17D dataset)
- reference m/z feature list (all detected features, e.g. 7,995 features)
- genome-scale metabolic model (KEGG, UCSD Recon1, Edinburgh human, or BioCyc)

## Outputs

- pathway enrichment table with Fisher's exact test overlap counts, EASE-score-corrected p-values and Gamma-modeled adjusted p-values
- JSON pathway analysis results for programmatic use

## How to apply

In mummichog, supply the significant feature list (selected by univariate statistics, e.g. t-test p-value cutoff) and the full reference list of all detected m/z features, plus a metabolic model (e.g. BioCyc human). The tool computes all possibly matched metabolites from m/z, then runs Fisher's exact test per pathway on the overlap between significant features and pathway metabolites; the EASE score is adopted to make the enrichment test more robust, and a permutation-based null p-value distribution is modeled as a Gamma distribution to assign an adjusted p-value per pathway. Output JSON pathway tables rank pathways by enrichment. Judgment signals: enriched pathways must be biologically coherent (e.g. glyoxylate shunt up in YF17D-activated cells), and the adjusted p-value ranking should reproduce expected biology from the validation dataset.

## Related tools

- **mummichog** (Core package that computes possibly matched metabolites from m/z, runs Fisher's exact test with EASE score per pathway, and fits a Gamma distribution to the permutation p-value null to assign adjusted p-values.) — http://atcg.googlecode.com
- **cloud-mummichog** (RESTful server/worker implementation of the same pathway enrichment pipeline, usable when the analysis is served from a web front-end.)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Adjusted p-values are derived from a Gamma-modeled permutation null, not a naive Fisher p-value alone — check the output JSON for the adjusted-p column.
- EASE-score correction is applied to the Fisher exact test overlap — the p-value should be more conservative than raw Fisher.
- Ranking of top pathways must be consistent across replicates/subsets of the feature table.
- For the validated urine dataset, predicted metabolites should reach 100% agreement (41/41) with the original annotation when the M+H[+] form is enforced.
- Feature counts entering the test must match the reference list size (e.g. 7,995 total features, 601 significant) — no silent filtering beyond the supplied lists.

## Limitations

- A single m/z feature may match several metabolites in the same pathway, inflating the overlap number.
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, so a de facto filtering occurs due to limited model coverage.
- The current version does not incorporate a statistical matrix from the feature selection step or mass flow balance of metabolic reactions.
- Quantification and feature selection, the steps upstream of this analysis, can still be problematic.
- Some metabolic models may lack pathway definitions; 'subsystem' can substitute as a pathway label.

## Evidence

- [results] Fisher's exact test with EASE score and Gamma-modeled adjusted p-values is the core statistical test.: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [results] mummichog requires two m/z lists.: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [results] Feature-count scale for the moDC YF17D dataset.: "yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were significantly different between the infected samples"
- [discussion] De facto filtering due to limited database annotation.: "the features from a high resolution profiling experiment by far exceed the current annotations in metabolite databases."
- [discussion] Feature-level ambiguity inflates overlap counts.: "Sometimes, a single m/z feature may match to several metabolites in the same module/pathway, inflating the overlap number."
- [readme] mummichog's role in the README: pathway/network analysis core package.: "mummichog(3): core algorithm package for pathway/network analysis"
- [results] mummichog computes possibly matched metabolites and searches the reference network.: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
