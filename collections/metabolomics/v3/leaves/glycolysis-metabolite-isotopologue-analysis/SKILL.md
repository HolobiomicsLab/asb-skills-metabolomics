---
name: glycolysis-metabolite-isotopologue-analysis
description: Use when quantified isotopologue peak areas for 13C6-glucose-traced glycolysis
  metabolites (e.g. G6P, F6P, FBP, DHAP, Pyr, Lac) from parallel tracer samples need
  to be converted to fractional labelling and checked against pathway position.
when_to_use_negative:
- Input is raw EIC data rather than already-quantified isotopologue peak areas — TarMet's
  isotopologue quantification step must run first.
- The tracer is not 13C6-glucose / the metabolites are not on the glycolysis pathway,
  so the upstream-labelled/downstream-unlabeled expectation does not apply.
- The goal is de novo tracer discovery or MS/MS library verification, which are listed
  as planned future features of TarMet, not current capability.
tools:
- name: TarMet
  role: source of the quantified isotopologue peak areas consumed by this fractional-labelling
    analysis
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: reactive GUI in TarMet through which targeted metabolite and isotopologue
    quantification is performed before fraction computation
  repo: https://github.com/hcji/TarMet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:8873829f36cc89c99d13085c678d9da9d292e340d27b1c226255d599e71ff0ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# glycolysis-metabolite-isotopologue-analysis

## Summary

Summarizes 13C6-glucose tracer isotopologue peak areas (from the TarMet quantification output) into fractional labelling per glycolysis metabolite and verifies the expected upstream-labelled / downstream-unlabeled pathway pattern.

## When to use

Use when quantified isotopologue peak areas for 13C6-glucose-traced glycolysis metabolites (e.g. G6P, F6P, FBP, DHAP, Pyr, Lac) from parallel tracer samples need to be converted to fractional labelling and checked against pathway position.

## When NOT to use

- Input is raw EIC data rather than already-quantified isotopologue peak areas — TarMet's isotopologue quantification step must run first.
- The tracer is not 13C6-glucose / the metabolites are not on the glycolysis pathway, so the upstream-labelled/downstream-unlabeled expectation does not apply.
- The goal is de novo tracer discovery or MS/MS library verification, which are listed as planned future features of TarMet, not current capability.

## Inputs

- quantified isotopologue peak areas for 13C6-glucose-traced glycolysis metabolites from the TarMet tracer workflow output (parallel samples)
- metabolite list with pathway positions (Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso)

## Outputs

- isotopologue_fractions.csv — summary table of fractional labelling (relative abundance per M+n isotopologue) for each glycolysis metabolite

## How to apply

Load the quantified isotopologue peak areas for the glycolysis metabolites produced by the upstream TarMet tracer workflow (TarMet quantifies targeted metabolites and their isotopologues via its reactive shiny GUI). For each metabolite, sum the areas of all its 13C isotopologues to obtain the total labelled pool, then divide each individual isotopologue area (M+1, M+2, ...) by the total to compute relative abundance (fractional labelling). Assemble these values for Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac and Cit/Iso into a summary table. Compare fractional labelling across pathway position: upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) should be predominantly labelled while downstream ones (Pyr, Ala, Lac) should remain unlabeled. Write the table as isotopologue_fractions.csv.

## Related tools

- **TarMet** (source of the quantified isotopologue peak areas consumed by this fractional-labelling analysis) — https://github.com/hcji/TarMet
- **shiny** (reactive GUI in TarMet through which targeted metabolite and isotopologue quantification is performed before fraction computation) — https://github.com/hcji/TarMet

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- For every metabolite, the fractional abundances of its isotopologues (M+1, M+2, ...) must sum to 1.
- Upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) are predominantly labelled while downstream ones (Pyr, Ala, Lac) remain unlabeled.
- Quantitative results across parallel samples are highly consistent.
- Output file isotopologue_fractions.csv contains one fractional-labelling row per metabolite for all 11 listed metabolites.

## Limitations

- TarMet currently lacks MS/MS library verification and data-independent acquisition support (planned future features), so isotopologue assignments cannot be independently verified within the tool.
- Validation relies on the expected metabolism rules (early labelled, late unlabeled); deviations need external follow-up.

## Evidence
