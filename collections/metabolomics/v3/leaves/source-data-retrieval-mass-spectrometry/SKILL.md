---
name: source-data-retrieval-mass-spectrometry
description: Use when reproducing a published MS-based claim from a paper whose figure-level
  underlying data is deposited as Source Data files, e.g. validating adduct abundance
  changes under salt infusion.
when_to_use_negative:
- The analysis starts from raw LC-MS/MS files — feature finding and IIMN must run
  first (MZmine/MS-DIAL/XCMS plus GNPS) before any Source Data table exists to compare
  against.
- Input is already a feature table without published deposited Source Data, so no
  figure-level statistics are available to reproduce.
- The target claim is about network connectivity or spectral library expansion rather
  than a quantitative abundance statistic tied to a specific figure.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MZmine
  role: produces the aligned feature list and ion-identity annotations whose deposited
    Source Data forms this skill's input; processing raw data with MZmine is the upstream
    alternative when raw files rather than Source Data are available
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: the environment in which the IIMN workflow that generated the deposited ion-identity
    abundances was run
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:2888c325d44777eeb0760a5e95b2436f3b616a85f7f4ad9b3ea1da91535733c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# source-data-retrieval-mass-spectrometry

## Summary

Retrieves and parses the deposited Source Data files accompanying published mass-spectrometry figures (e.g. the post-column salt-infusion benchmark underlying Figs. 2–3 of the IIMN paper) so that ion-identity abundance statistics can be reproduced statistically.

## When to use

Use when reproducing a published MS-based claim from a paper whose figure-level underlying data is deposited as Source Data files, e.g. validating adduct abundance changes under salt infusion.

## When NOT to use

- The analysis starts from raw LC-MS/MS files — feature finding and IIMN must run first (MZmine/MS-DIAL/XCMS plus GNPS) before any Source Data table exists to compare against.
- Input is already a feature table without published deposited Source Data, so no figure-level statistics are available to reproduce.
- The target claim is about network connectivity or spectral library expansion rather than a quantitative abundance statistic tied to a specific figure.

## Inputs

- Source Data files (tabular) accompanying published MS figures
- ion-identity annotation table with adduct labels ([M+Na]+, [M+NH4]+)
- per-sample feature abundance table
- condition/infusion metadata mapping samples to H2O / Na-acetate / NH4-acetate

## Outputs

- per-adduct mean abundances per infusion condition
- fold changes versus control
- Welch t-test statistics and p-values per salt-infusion comparison
- tabulated reproduction of reported figure-level statistics

## How to apply

Download the Source Data files deposited with the target figures — in the IIMN (Nature Communications 2021) paper these accompany the post-column salt-infusion benchmark of a 300-compound mixture infused with H2O, sodium acetate, or ammonium acetate, and support Figs. 2–3. Parse the ion-identity annotation and per-sample abundance tables, extracting features annotated as [M+Na]+ or [M+NH4]+ together with their control (H2O infusion) counterparts, as annotated by the Ion Identity Molecular Networking (IIMN) workflow. Group abundances by infusion condition, then run Welch two-sample t-tests (unequal variances) comparing, for example, [M+Na]+ abundances under Na-acetate infusion against control. The rationale is that the salt-infusion experiment constitutes a ground truth dataset with induced adduct formation, so statistically significant abundance shifts of the corresponding adducts validate IIMN ion-species identification at the dataset level; significance is judged at p < 0.05 (the article reports p < 0.001). Tabulate per-adduct mean abundances, fold changes, t-statistics, and p-values for each salt-infusion-versus-control comparison.

## Related tools

- **MZmine** (produces the aligned feature list and ion-identity annotations whose deposited Source Data forms this skill's input; processing raw data with MZmine is the upstream alternative when raw files rather than Source Data are available) — https://github.com/mzmine/mzmine
- **GNPS** (the environment in which the IIMN workflow that generated the deposited ion-identity abundances was run) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
gnps download --task-id source_data_fig2_3 --output salt_infusion_benchmark/ ; python parse_and_test.py --input salt_infusion_benchmark/*.csv --adducts [M+Na]+,[M+NH4]+ --test welch
```

## Evaluation signals

- Reproduced Welch test for [M+Na]+ under Na-acetate infusion vs control must show a significant abundance increase (p < 0.05; article reports p < 0.001)
- Reproduced Welch test for [M+NH4]+ under NH4-acetate infusion vs control must show a significant abundance increase (p < 0.05; article reports p < 0.001)
- Control (H2O) group abundances must match the deposited Source Data values row-for-row after parsing
- Per-adduct fold changes should be directionally consistent (increase under the corresponding salt infusion, not decrease)
- Tabulated means and t-statistics should match the Source Data-reported figure statistics within numerical tolerance

## Limitations

- No raw data accession ID for the post-column salt infusion experiments is stated in the section text, so retrieval depends on the publisher's supplementary/Source Data availability
- No code repository for the salt infusion analysis or the statistical extraction workflow is given — the statistical workflow must be re-implemented
- The article mentions a 'special MZmine IIMN build' whose release status is unclear; reproducing raw-data-to-feature steps may not be possible from text alone
- Only figure-level aggregated data may be deposited; per-sample raw abundances are not guaranteed to be present in every Source Data file

## Evidence

- [intro] post-column infusion of salt solutions ground truth: [evidence span withheld — anchor status: not_found]
- [abstract] IIMN integrates chromatographic peak shape correlation into molecular networks: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [discussion] three widely used open source MS processing tools (MZmine17, MS-DIAL18, and XCMS13,16): "we interfaced the IIMN workflow with three widely used open source MS processing tools (MZmine17, MS-DIAL18, and XCMS13,16)"
- [discussion] The special MZmine IIMN build offers two modules in the Tools tab: "The special MZmine IIMN build offers two modules in the Tools tab"
- [discussion] [no raw data accession for salt infusion experiments stated in the section text]: [evidence span withheld — anchor status: not_found]
- [discussion] upstream feature finding produces the aligned feature list: "Process the data to create a feature list (aligned overall samples)"
