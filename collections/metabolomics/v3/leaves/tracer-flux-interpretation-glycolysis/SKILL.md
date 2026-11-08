---
name: tracer-flux-interpretation-glycolysis
description: Use when 13C6-glucose tracer isotopologue quantification (e.g., TarMet
  peak areas) is available and the goal is to check parallel-sample consistency and
  the upstream-labelled/downstream-unlabeled pattern along glycolysis.
when_to_use_negative:
- Input is raw LC-MS data rather than already-quantified isotopologue areas from the
  TarMet tracer workflow — use TarMet's quantification/isotopologue analysis steps
  first
- The tracer or pathway differs (not 13C6-glucose tracing of the glycolysis cascade),
  so the upstream-labelled/downstream-unlabeled expectation does not apply
- DIA data or MS/MS library verification is required — these were listed in TarMet
  as planned future features, not supported
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TarMet
  role: Upstream tool that quantifies targeted metabolites and their isotopologues;
    its tracer workflow output provides the isotopologue peak areas this skill interprets
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Reactive GUI framework through which TarMet's targeted metabolite and isotopologue
    quantification is operated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:708be989910dc60f35175df70274a8698e4a3b048a06de03c7a2a0a83795f3b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - shiny
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# tracer-flux-interpretation-glycolysis

## Summary

Interprets quantified 13C isotopologue peak areas from a TarMet stable-isotope tracer workflow by computing fractional labelling per metabolite and validating the expected glycolysis pathway labelling pattern (upstream metabolites labelled, downstream ones unlabeled).

## When to use

Use when 13C6-glucose tracer isotopologue quantification (e.g., TarMet peak areas) is available and the goal is to check parallel-sample consistency and the upstream-labelled/downstream-unlabeled pattern along glycolysis.

## When NOT to use

- Input is raw LC-MS data rather than already-quantified isotopologue areas from the TarMet tracer workflow — use TarMet's quantification/isotopologue analysis steps first
- The tracer or pathway differs (not 13C6-glucose tracing of the glycolysis cascade), so the upstream-labelled/downstream-unlabeled expectation does not apply
- DIA data or MS/MS library verification is required — these were listed in TarMet as planned future features, not supported

## Inputs

- Quantified isotopologue peak areas for 13C6-glucose-traced metabolites from the upstream TarMet tracer workflow output
- Metabolite-to-pathway-position mapping for glycolysis metabolites (Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso)

## Outputs

- isotopologue_fractions.csv containing per-metabolite fractional labelling (relative abundance) for each 13C isotopologue (M+1, M+2, ...)
- A validation check that fractional abundances sum to 1 per metabolite and that the expected labelling pattern across pathway position holds

## How to apply

Load the quantified isotopologue peak areas for glycolysis metabolites from the upstream TarMet tracer workflow output. For each metabolite, sum the areas of all its 13C isotopologues to obtain the total labelled pool, then divide each individual isotopologue area (M+1, M+2, ...) by this total to get the relative fractional labelling per isotopologue. Assemble the fractional labelling values for Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac and Cit/Iso into a summary table. Verify that each metabolite's isotopologue fractional abundances sum to 1, that parallel samples are highly consistent, and that upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) are predominantly labelled while downstream ones (Pyr, Ala, Lac) remain unlabeled, confirming the metabolism rules. Write the result as isotopologue_fractions.csv.

## Related tools

- **TarMet** (Upstream tool that quantifies targeted metabolites and their isotopologues; its tracer workflow output provides the isotopologue peak areas this skill interprets) — https://github.com/hcji/TarMet
- **shiny** (Reactive GUI framework through which TarMet's targeted metabolite and isotopologue quantification is operated)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- For every metabolite, the fractional abundances of its isotopologues sum to 1
- Fractional labelling values are highly consistent across parallel (replicate) samples
- Upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) show predominantly labelled isotopologues, while downstream metabolites (Pyr, Ala, Lac) remain unlabeled
- The summary table covers all expected metabolites (Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso) and is written as isotopologue_fractions.csv

## Limitations

- Depends on the quality of the upstream TarMet isotopologue quantification; TarMet is reported to detect more isotopologues and quantify them better than state-of-the-art methods, but errors there propagate
- Theoretical isotope patterns used in quantification depend on the MS resolution via the enviPat package
- TarMet does not yet support data-independent acquisition or MS/MS library verification (planned future features)

## Evidence
