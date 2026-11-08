---
name: metabolomics-data-summarization
description: Use when TarMet tracer quantification has already produced per-sample
  isotopologue peak areas (e.g. for 13C6-glucose-traced glycolysis metabolites such
  as G6P, F6P, FBP, DHAP, Pyr, Lac) and you need a normalized fractional-labelling
  summary to verify pathway labelling patterns.
when_to_use_negative:
- Input is raw LC-MS data or EIC chromatograms rather than already-quantified isotopologue
  peak areas from the TarMet tracer workflow — quantification must happen first.
- The metabolites are not isotopologue-resolved targets (no M+n labelling information),
  e.g. unlabelled targeted or untargeted feature tables.
- Only a single sample without parallel replicates is available, so cross-sample consistency
  of labelling cannot be assessed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TarMet
  role: shiny application that quantified the targeted metabolites and their isotopologues;
    its tracer workflow output (isotopologue peak areas) is the direct input for the
    fractional-labelling summary
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: provides the reactive graphical user interface through which the TarMet quantification
    and isotopologue analysis are run before summarization
  repo: https://github.com/hcji/TarMet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:9dd4aff7f21ba901a95d0d047482019a794d52900c79a79f3b7b27f25858fdbe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# metabolomics-data-summarization

## Summary

Summarizes quantified 13C isotopologue peak areas from a stable-isotope tracer experiment into per-metabolite fractional labelling values (relative isotopologue abundances), assembled into a pathway-position summary table that reveals where label accumulates along a metabolic route.

## When to use

Use when TarMet tracer quantification has already produced per-sample isotopologue peak areas (e.g. for 13C6-glucose-traced glycolysis metabolites such as G6P, F6P, FBP, DHAP, Pyr, Lac) and you need a normalized fractional-labelling summary to verify pathway labelling patterns.

## When NOT to use

- Input is raw LC-MS data or EIC chromatograms rather than already-quantified isotopologue peak areas from the TarMet tracer workflow — quantification must happen first.
- The metabolites are not isotopologue-resolved targets (no M+n labelling information), e.g. unlabelled targeted or untargeted feature tables.
- Only a single sample without parallel replicates is available, so cross-sample consistency of labelling cannot be assessed.

## Inputs

- quantified isotopologue peak areas (M+0, M+1, M+2, ...) for tracer metabolites from the upstream TarMet tracer workflow
- list of target pathway metabolites (e.g. Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso)
- parallel/replicate sample grouping information

## Outputs

- isotopologue_fractions.csv — table of per-metabolite fractional isotopologue abundances (M+1, M+2, ... fractions summing to 1)
- pathway-position comparison showing upstream metabolites predominantly labelled and downstream metabolites unlabeled

## How to apply

Load the quantified isotopologue peak areas for the tracer-relevant metabolites from the upstream TarMet tracer workflow output. For each metabolite, sum the areas of all its 13C isotopologues to obtain the total labelled pool, then divide each individual 13C isotopologue area (M+1, M+2, ...) by that total to compute relative abundance (fractional labelling) per isotopologue. Assemble the fractional labelling values for the pathway metabolites (e.g. Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso) into a summary table and compare fractional labelling across pathway position: upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) should be predominantly labelled while downstream ones (Pyr, Ala, Lac) should remain largely unlabeled. Write the resulting table as isotopologue_fractions.csv. The rationale is that fractional (normalized) labelling removes injection-quantity effects and makes parallel samples directly comparable, exposing the expected metabolism rules of the tracer.

## Related tools

- **TarMet** (shiny application that quantified the targeted metabolites and their isotopologues; its tracer workflow output (isotopologue peak areas) is the direct input for the fractional-labelling summary) — https://github.com/hcji/TarMet
- **shiny** (provides the reactive graphical user interface through which the TarMet quantification and isotopologue analysis are run before summarization) — https://github.com/hcji/TarMet

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- For every metabolite, the fractional abundances of all its isotopologues (M+1, M+2, ...) must sum to 1 (within numerical tolerance).
- Quantitative results of parallel samples must be highly consistent (replicate agreement in fractional labelling).
- Earlier metabolites in the pathway must be predominantly labelled while later ones remain unlabeled — the expected pattern from 13C6-glucose tracing.
- The output file isotopologue_fractions.csv exists and contains one fractional-labelling row/set per target metabolite (Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso).

## Limitations

- Interpretation depends on the correctness of the upstream TarMet isotopologue quantification; TarMet's future-planned MS/MS library verification and data-independent acquisition support were not yet implemented at time of publication.
- Labelling-pattern conclusions assume the tracer experiment's expected metabolism rules (upstream labelled, downstream unlabeled); deviations require biological rather than purely computational interpretation.
- The summarization is per-metabolite fractional normalization and does not itself correct for natural isotope abundance (theoretical isotope patterns are handled upstream by enviPat within TarMet).

## Evidence

- [results] finding: parallel samples consistent, upstream labelled: [evidence span withheld — anchor status: not_found]
- [full_text] workflow: compute fractional labelling per isotopologue: [evidence span withheld — anchor status: not_found]
