---
name: stable-isotope-fractional-labelling-calculation
description: Use when quantified 13C isotopologue peak areas from a stable isotope
  tracer experiment (e.g., 13C6-glucose) are available and the goal is fractional
  labelling per metabolite to check pathway labelling patterns across parallel samples.
when_to_use_negative:
- Input is raw LC-MS data or extracted-ion chromatograms rather than already-quantified
  isotopologue peak areas (run TarMet's isotopologue analysis/quantification first).
- The analysis targets unlabelled targeted metabolite quantification where no stable
  isotope tracer was used, so fractional labelling is undefined.
- Tracer design does not produce interpretable positional labelling (e.g., metabolites
  outside the traced pathway), so the upstream-labelled/downstream-unlabeled check
  does not apply.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: TarMet
  role: Shiny application used to quantify targeted metabolites and their isotopologues,
    producing the isotopologue areas consumed by the fractional-labelling calculation;
    also usable via R scripts from the command line vignette.
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Provides the reactive graphical user interface through which the TarMet isotopologue
    quantification and tracer analysis are driven.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:d9ccb67cbb8989e8011091b8380dce39158a540baa91f63e21fdff0851b98581
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - shiny
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# stable-isotope-fractional-labelling-calculation

## Summary

Compute per-metabolite fractional 13C labelling from TarMet-quantified isotopologue peak areas by normalizing each isotopologue (M+1, M+2, ...) to the summed labelled pool, enabling pathway-order verification in tracer experiments such as 13C6-glucose tracing of glycolysis.

## When to use

Use when quantified 13C isotopologue peak areas from a stable isotope tracer experiment (e.g., 13C6-glucose) are available and the goal is fractional labelling per metabolite to check pathway labelling patterns across parallel samples.

## When NOT to use

- Input is raw LC-MS data or extracted-ion chromatograms rather than already-quantified isotopologue peak areas (run TarMet's isotopologue analysis/quantification first).
- The analysis targets unlabelled targeted metabolite quantification where no stable isotope tracer was used, so fractional labelling is undefined.
- Tracer design does not produce interpretable positional labelling (e.g., metabolites outside the traced pathway), so the upstream-labelled/downstream-unlabeled check does not apply.

## Inputs

- Quantified 13C isotopologue peak areas for tracer metabolites (TarMet tracer workflow output)
- List of metabolites ordered by pathway position (e.g., G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso)

## Outputs

- isotopologue_fractions.csv — summary table of fractional labelling (relative isotopologue abundances) per metabolite per sample

## How to apply

Load the quantified isotopologue peak areas for tracer metabolites from the upstream TarMet tracer workflow output. For each metabolite, sum the areas of all its 13C isotopologues to obtain the total labelled pool, then divide each individual isotopologue area (M+1, M+2, ...) by that total to obtain the relative abundance (fractional labelling) per isotopologue. Assemble the fractional labelling values for the metabolites of interest (e.g., Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso) into a summary table and write it as isotopologue_fractions.csv. The rationale is that fractional normalization makes parallel samples directly comparable and exposes pathway position: upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) should be predominantly labelled while downstream ones (Pyr, Ala, Lac) remain unlabeled, confirming metabolism rules. Validate that fractional abundances of each metabolite's isotopologues sum to 1 and that the expected upstream-labelled/downstream-unlabeled pattern is reproduced.

## Related tools

- **TarMet** (Shiny application used to quantify targeted metabolites and their isotopologues, producing the isotopologue areas consumed by the fractional-labelling calculation; also usable via R scripts from the command line vignette.) — https://github.com/hcji/TarMet
- **shiny** (Provides the reactive graphical user interface through which the TarMet isotopologue quantification and tracer analysis are driven.)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- For every metabolite, the fractional abundances of its isotopologues (M+1, M+2, ...) sum to 1.
- Fractional labelling results from parallel samples are highly consistent (replicate agreement).
- Upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) are predominantly labelled while downstream ones (Pyr, Ala, Lac) remain unlabeled, matching the expected 13C6-glucose pathway labelling pattern.
- isotopologue_fractions.csv is produced containing fractional labelling for all requested metabolites (Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso).

## Limitations

- Pattern interpretation assumes the tracer enters at a known pathway position; earlier-vs-later labelling logic is only valid for the traced pathway.
- TarMet's stated future work includes MS/MS library verification and data-independent acquisition support, so isotopologue identity currently relies on the targeted metabolite definition and theoretical isotope patterns from enviPat at the given MS resolution.
- The fractional-labelling check depends on quality of upstream peak detection (MassSpecWavelet), baseline correction (airPLS), and retention-time alignment (PAFFT); errors there propagate into isotopologue areas.

## Evidence

- [methods] Fractional labelling workflow from source card: [evidence span withheld — anchor status: not_found]
- [methods] Validation rule from source card: [evidence span withheld — anchor status: not_found]
- [results] Tracer finding from source card: [evidence span withheld — anchor status: not_found]
