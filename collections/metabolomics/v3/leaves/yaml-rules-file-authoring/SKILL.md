---
name: yaml-rules-file-authoring
description: Use when suspect screening results (formula and compound annotations
  with scores and candidate rankings) need automatic, rule-based estimation of Schymanski
  identification levels and the mapping from scores/labels to levels must be inspectable
  or customizable.
when_to_use_negative:
- The input is raw MS data or a feature table without formula/compound structure annotation
  — levels cannot be estimated before annotation exists
- The workflow uses patRoon only for TP screening, blank subtraction or feature extraction
  without suspect screening annotations to classify
- Manual expert assessment of individual suspect hits is required (e.g. for definitive
  confirmation with a reference standard) rather than rule-based estimation
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: patRoon 2.0
  role: Provides genIDLevelRulesFile() to generate the YAML rule template and estimateIDConfidence()
    with the IDFile argument to apply the rules to formula/compound annotations
  repo: https://rickhelmus.github.io/patRoon
- name: R
  role: Runtime environment in which patRoon's rule-generation and confidence-estimation
    functions are called
  repo: https://rickhelmus.github.io/patRoon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:ac692ba860a83db699b12a16e795372b546f0f20c95f6ed38291d03408280eaa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - patRoon 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# yaml-rules-file-authoring

## Summary

Authors and applies a configurable YAML rule file that automatically assigns Schymanski et al. (2014) identification levels (1–5) to suspect-screening annotations in patRoon 2.0, replacing manual confidence assessment of formula and compound structure candidates.

## When to use

Use when suspect screening results (formula and compound annotations with scores and candidate rankings) need automatic, rule-based estimation of Schymanski identification levels and the mapping from scores/labels to levels must be inspectable or customizable.

## When NOT to use

- The input is raw MS data or a feature table without formula/compound structure annotation — levels cannot be estimated before annotation exists
- The workflow uses patRoon only for TP screening, blank subtraction or feature extraction without suspect screening annotations to classify
- Manual expert assessment of individual suspect hits is required (e.g. for definitive confirmation with a reference standard) rather than rule-based estimation

## Inputs

- Formula annotation results for features (e.g. from GenForm/SIRIUS)
- Compound structure annotation results for features (e.g. from MetFrag/SIRIUS)
- YAML rules file (e.g. idlevelrules.yml) generated or edited via genIDLevelRulesFile()

## Outputs

- Annotation objects extended with an identification-level column (values 1–5 per Schymanski et al., 2014)
- Generated/configured YAML rule file (idlevelrules.yml)
- Feature table with candidates, scores and assigned identification levels (e.g. identification_levels.csv)

## How to apply

First generate a template with genIDLevelRulesFile(), which writes the default YAML rule set (e.g. idlevelrules.yml) that maps annotation scores, rank-1 candidate status and supporting evidence such as reference-standard or MS/MS matches to identification levels. Open and adjust the YAML thresholds so levels follow the Schymanski et al. (2014) convention (levels 1–5); the file is deliberately configurable and extensible so the rules can be tuned per project. Then run estimateIDConfidence() passing the formula and compound annotation results together with the YAML file via the IDFile argument, which evaluates each annotated feature against the rules and attaches an identification level. The rationale is that manual curation of thousands of suspect hits is impractical, while a transparent rules file keeps the level assignment reproducible and auditable.

## Related tools

- **patRoon 2.0** (Provides genIDLevelRulesFile() to generate the YAML rule template and estimateIDConfidence() with the IDFile argument to apply the rules to formula/compound annotations) — https://rickhelmus.github.io/patRoon
- **R** (Runtime environment in which patRoon's rule-generation and confidence-estimation functions are called) — https://rickhelmus.github.io/patRoon

## Examples

```
rulesFile <- genIDLevelRulesFile("idlevelrules.yml"); annotations <- estimateIDConfidence(annotations, IDFile = rulesFile)
```

## Evaluation signals

- The generated YAML rules file exists and is parseable (e.g. valid YAML with threshold entries for scores, candidate ranks, MS/MS matches)
- identification_levels.csv exists and contains an identification-level column populated for all annotated candidates
- All assigned identification levels fall within the 1–5 range of the Schymanski et al. (2014) convention
- Level assignments are reproducible: re-running estimateIDConfidence() with the same YAML file yields identical levels
- Rank-1 candidates with strong supporting evidence receive higher levels than lower-ranked or weakly supported candidates, consistent with the configured thresholds

## Limitations

- The estimated levels are only as correct as the configured rule thresholds; the YAML must be inspected and adjusted to truly follow the Schymanski convention
- Level estimation applies to formula and compound annotations produced within patRoon's interface; results from unannotated features cannot be classified
- The rules encode configurable thresholds, not definitive chemical confirmation — a high rule-derived level does not replace reference-standard confirmation

## Evidence

- [abstract] Improved suspect screening includes automatic estimation of identification levels using a configurable rule-based approach: "Automatic estimation of identification levels (Schymanski et al., 2014) using a configurable and extensible rule based approach"
- [readme] patRoon includes novel ID-level estimation functionality: [evidence span withheld — anchor status: external_doc]
- [readme] README functionality table lists ID confidence as native capability: [evidence span withheld — anchor status: external_doc]
