---
name: permutation-null-pvalue-gamma-modeling
description: Use when enrichment p-values from Fisher's exact tests on network module/pathway
  hits need a permutation-estimated null because the small-of overlap-corrected overlap
  count is non-standard.
when_to_use_negative:
- When standard FET assumptions hold and overlaps are unambiguous (no m/z-to-multiple-metabolite
  matching), a plain Fisher's exact test p-value may suffice without permutation.
- When the input is already annotated metabolites rather than unassigned m/z features,
  standard gene-set-style ORA with an empirical null may be preferable to mummichog's
  m/z-permutation null.
- When the statistical test upstream (feature selection) is itself unvalidated or
  multivariate, the permutation null only characterizes enrichment noise, not upstream
  quantification error.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mummichog
  role: core algorithm package that performs the permutation of m/z features and fits
    a Gamma curve to the null p-value distribution for pathway/module enrichment
  repo: http://atcg.googlecode.com
- name: cloud-mummichog
  role: RESTful server/worker implementation that invokes the same permutation/Gamma
    enrichment analysis
  repo: metabolomics-cloud/mummichog
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:32c70b634a3a590665c6c780c8a4848d72dfdb92f29160107f13f5ef71d2545d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - cloud-mummichog
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# Permutation-null p-value Gamma modeling

## Summary

Estimates the null p-value distribution of pathway/module enrichment statistics in mummichog by random permutation of m/z features and fitting a Gamma curve, yielding adjusted p-values that account for the non-standard null implied by m/z-based metabolite matching.

## When to use

Use when enrichment p-values from Fisher's exact tests on network module/pathway hits need a permutation-estimated null because the small-of overlap-corrected overlap count is non-standard.

## When NOT to use

- When standard FET assumptions hold and overlaps are unambiguous (no m/z-to-multiple-metabolite matching), a plain Fisher's exact test p-value may suffice without permutation.
- When the input is already annotated metabolites rather than unassigned m/z features, standard gene-set-style ORA with an empirical null may be preferable to mummichog's m/z-permutation null.
- When the statistical test upstream (feature selection) is itself unvalidated or multivariate, the permutation null only characterizes enrichment noise, not upstream quantification error.

## Inputs

- List of significant m/z features (m/z, rtime, p-value) from a statistical test
- Reference list of all m/z features detected in the MS experiment
- Genome-scale metabolic model (list_of_pathways, list_of_reactions, list_of_compounds, with neutral chemical formulas)

## Outputs

- Gamma-modeled null p-value distribution for pathway enrichment
- Per-pathway adjusted p-values and result tables (result.html in mummichog v2)
- JSON strings of pathway analysis results for programmatic use (mummichog v3)

## How to apply

Run mummichog's pathway analysis after tentatively matching feature m/z values to metabolites via adduct and isotope enumeration. The basic statistic is Fisher's exact test on each pathway's overlap with the significant feature list, but a single m/z feature can match several metabolites in the same module, inflating the overlap count; mummichog therefore applies the 'smaller-of' correction. Rather than assuming a theoretical FET null, generate a null p-value distribution by random permutations of m/z features and model it as a Gamma curve. Compute EASE score (the smaller-of correction) for robustness, then derive a p-value per pathway/module against the Gamma-modeled null and report adjusted p-values in the result tables. The rationale, stated in the article, is that 'the statistics on the list of up regulated compounds ... are similar in spirit to an over-representation analysis (ORA)', which requires an empirical null rather than a textbook FET distribution.

## Related tools

- **mummichog** (core algorithm package that performs the permutation of m/z features and fits a Gamma curve to the null p-value distribution for pathway/module enrichment) — http://atcg.googlecode.com
- **cloud-mummichog** (RESTful server/worker implementation that invokes the same permutation/Gamma enrichment analysis) — metabolomics-cloud/mummichog

## Examples

```
python3 -m mummichog.main -f mummichog/tests/testdata0710.txt -o t3
```

## Evaluation signals

- Null p-value distribution should follow a Gamma shape rather than the uniform distribution expected under a naive FET null; check the fitted Gamma parameters are non-degenerate.
- Adjusted p-values should be ≥ the raw Fisher's exact test p-values for the same pathway, reflecting the inflation correction for m/z-to-multiple-metabolite overlaps.
- Pathways with known biological activity (e.g. in the moDC/YF17D activation dataset) should rank with significant adjusted p-values in result.html.
- Number of significant features fed into the permutation (601 in the moDC dataset out of 7,995 total) should match the feature table supplied.
- Re-running permutations with a different random seed should produce stable adjusted p-value ranks (robustness of the Gamma fit).

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number; the smaller-of (EASE) correction is a heuristic fix, not an exact solution.
- The current mummichog version does not incorporate a statistical matrix from the feature selection step prior to enrichment analysis, so upstream quantification errors are not propagated into the null.
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, so the effective overlap counts are limited by annotation coverage, which the permutation null partially accounts for but does not eliminate.

## Evidence

- [results] Fisher's exact test with permutation-based null p-value distribution modeled as Gamma: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [results] Permutation-based null p-value distribution modeled as Gamma: [evidence span withheld — anchor status: not_found]
- [discussion] Single m/z feature matching multiple metabolites inflates overlap: "Sometimes, a single m/z feature may match to several metabolites in the same module/pathway, inflating the overlap number."
- [results] EASE score / smaller-of correction for robustness: [evidence span withheld — anchor status: not_found]
- [readme] Feature table input requirement: "User supplied features with m/z, rtime, p-value from a statistical test."
- [readme] Output format includes result tables and JSON strings: "Outpout are 1. Result tables and figures, result.html as ver 2. 2. JSON strings from pathway analysis and network module analysis for programmatic use."
- [results] moDC dataset used for validation of the enrichment pipeline: "yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were significantly different between the infected samples"
