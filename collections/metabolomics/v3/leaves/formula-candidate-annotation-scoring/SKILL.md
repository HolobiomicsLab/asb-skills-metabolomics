---
name: formula-candidate-annotation-scoring
description: Use when suspect screening results from patRoon (formula candidates from
  GenForm/SIRIUS, compound candidates from MetFrag/SIRIUS) need automatic Schymanski
  identification-level assignment that is reproducible and rule-driven.
when_to_use_negative:
- Input is a raw feature table without formula or compound structure annotations —
  the rules score annotation evidence, so no candidates means nothing to estimate
  levels from
- The analysis targets transformation product screening or feature grouping (componentization/adduct
  annotation) rather than assigning confidence levels to suspect screening hits
- Levels must be assigned by manual expert curation or by a published scoring scheme
  incompatible with the Schymanski et al. (2014) level convention
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: patRoon 2.0
  role: Provides genIDLevelRulesFile() to generate the YAML rule template and estimateIDConfidence()
    to score formula/compound annotations into Schymanski levels
  repo: https://github.com/rickhelmus/patRoon
- name: R
  role: Platform in which patRoon's identification-level estimation workflow is scripted
    and executed
- name: patRoonData
  role: Supplies the example standard-X/solvent-X LC-HRMS data used to exercise and
    validate the level-estimation workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.04029
    title: 'patRoon 2.0: Improved non-target analysis workflows including automated
      transformation product screening'
  source_content_hash: sha256:01bc0f8f1d039baa3bc950f802d6e51c1288cfe9fc78ecd435d6a06ebd0f710a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - R
  - patRoon 2.0
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# formula-candidate-annotation-scoring

## Summary

A patRoon 2.0 method that automatically assigns Schymanski et al. (2014) suspect identification levels (1-5) to LC-HRMS features using formula and compound structure annotation scores, candidate rankings and supporting data, driven by a configurable and extensible YAML rule file.

## When to use

Use when suspect screening results from patRoon (formula candidates from GenForm/SIRIUS, compound candidates from MetFrag/SIRIUS) need automatic Schymanski identification-level assignment that is reproducible and rule-driven.

## When NOT to use

- Input is a raw feature table without formula or compound structure annotations — the rules score annotation evidence, so no candidates means nothing to estimate levels from
- The analysis targets transformation product screening or feature grouping (componentization/adduct annotation) rather than assigning confidence levels to suspect screening hits
- Levels must be assigned by manual expert curation or by a published scoring scheme incompatible with the Schymanski et al. (2014) level convention

## Inputs

- patRoon formula annotation results (candidates calculated by GenForm or SIRIUS)
- patRoon compound structure annotation results (candidates from MetFrag, SIRIUS or native annotation)
- YAML rule file mapping scores, candidate rankings and supporting data to identification levels (e.g. idlevelrules.yml, generated with genIDLevelRulesFile())
- patRoonData LC-HRMS example data (standard-X standard mixture and solvent-X blank, triplicate, m/z 75-450)

## Outputs

- Identification level (1-5 per Schymanski et al., 2014) attached to every annotated feature candidate
- Feature table with candidates, scores and assigned identification levels (e.g. identification_levels.csv)
- Generated/curated YAML rules file

## How to apply

First generate a template rule file with genIDLevelRulesFile(), which writes a YAML file (e.g. idlevelrules.yml) containing the default rule set that maps annotation scores, candidate rankings and supporting data (such as presence of a reference standard or MS/MS match) to identification levels. Inspect and adjust rule thresholds so the mapping follows the Schymanski et al. (2014) convention. Load the formula and compound annotation results for the features of interest (e.g. the patRoonData standard-X / solvent-X LC-HRMS example data), then call estimateIDConfidence() with the annotation results and the YAML IDFile so that every annotated feature receives a level in the 1-5 range. The rule-based approach is configurable and extensible, so custom evidence types can be added rather than hard-coding ad-hoc criteria.

## Related tools

- **patRoon 2.0** (Provides genIDLevelRulesFile() to generate the YAML rule template and estimateIDConfidence() to score formula/compound annotations into Schymanski levels) — https://github.com/rickhelmus/patRoon
- **R** (Platform in which patRoon's identification-level estimation workflow is scripted and executed)
- **patRoonData** (Supplies the example standard-X/solvent-X LC-HRMS data used to exercise and validate the level-estimation workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
idLevels <- estimateIDConfidence(formulaAnn, compoundAnn, IDFile = "idlevelrules.yml")
```

## Evaluation signals

- identification_levels.csv exists and contains an identification-level column with values in the 1-5 range for all annotated candidates
- The generated YAML rules file (e.g. idlevelrules.yml) is present and parseable
- Every annotated feature in the returned annotation object carries a non-missing identification level
- Rank-1 candidates with strong annotation scores and MS/MS or reference-standard support map to better (lower-number) levels than candidates without such support, consistent with Schymanski et al. (2014)

## Limitations

- Assigned levels are only as good as the underlying formula/compound annotations; rule-based estimation cannot compensate for missing annotation scores or absent MS/MS data
- Default rule thresholds are a starting point and must be inspected/adjusted (e.g. minimum score for a rank-1 candidate) to genuinely reflect the Schymanski et al. (2014) convention
- The automatic estimation is specific to the Schymanski et al. (2014) identification-level scale and patRoon's annotation data structures

## Evidence

- [abstract] abstract: "Automatic estimation of identification levels (Schymanski et al., 2014) using a configurable and extensible rule based approach"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
