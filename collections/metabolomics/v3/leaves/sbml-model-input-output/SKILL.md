---
name: sbml-model-input-output
description: Use when a genome-scale community metabolic model built with PyCoMo (e.g.
  the three-species biogas model of D. vulgaris, M. hungatei, M. barkeri) must be
  persisted to, or reloaded from, an SBML file for sharing, reuse, or downstream COBRA
  analysis.
when_to_use_negative:
- The goal is analysis (exchange metabolite/cross-feeding prediction, max growth rate,
  FVA) rather than persisting or reusing the model — use PyCoMo's analysis workflows
  instead.
- The input is a single-organism metabolic model with no community structure — there
  is no shared exchange compartment or member-prefixed IDs for this skill to preserve.
- The model was not generated with PyCoMo and lacks the expected SBML conventions
  (shared exchange compartment named 'medium' or a shared_compartment_id parameter)
  required by downstream consumers such as ScyNet.
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: PyCoMo
  role: Generates and saves/loads the community metabolic models in SBML format
  repo: http://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Reloads the saved community SBML model and applies further COBRA methods directly
- name: SBML
  role: Serialization format used to save and reuse PyCoMo community models
- name: openCOBRA
  role: File-format compliance target for the generated community models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:a6a6314ae4783026e441505e3d224ff0ad15d84789ff3a0c2fe3c160a4da6440
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - SBML
  - openCOBRA
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sbml-model-input-output

## Summary

Saving and reloading PyCoMo-generated compartmentalized community metabolic models in SBML format so they remain COBRApy-compatible, fully reusable, and retain member-of-origin attribution and original reaction bounds.

## When to use

Use when a genome-scale community metabolic model built with PyCoMo (e.g. the three-species biogas model of D. vulgaris, M. hungatei, M. barkeri) must be persisted to, or reloaded from, an SBML file for sharing, reuse, or downstream COBRA analysis.

## When NOT to use

- The goal is analysis (exchange metabolite/cross-feeding prediction, max growth rate, FVA) rather than persisting or reusing the model — use PyCoMo's analysis workflows instead.
- The input is a single-organism metabolic model with no community structure — there is no shared exchange compartment or member-prefixed IDs for this skill to preserve.
- The model was not generated with PyCoMo and lacks the expected SBML conventions (shared exchange compartment named 'medium' or a shared_compartment_id parameter) required by downstream consumers such as ScyNet.

## Inputs

- PyCoMo-generated compartmentalized community metabolic model (COBRApy model in memory)
- Member genome-scale metabolic models in SBML format (e.g. D. vulgaris, M. hungatei, M. barkeri from Koch et al. 2019)

## Outputs

- community_model.sbml — SBML file of the community model, reloadable as a COBRApy model
- Reloaded COBRApy model preserving member-of-origin attribution and original reaction bounds

## How to apply

After generating the compartmentalized community model with PyCoMo (member models loaded from genome-scale SBML files, exchange metabolites matched by identical IDs, medium configured with e.g. ethanol and CO2), save it in SBML format (e.g. community_model.sbml). Because PyCoMo community models are COBRApy models, the saved file reloads directly with COBRApy and can be used by other COBRA methods without conversion. When reloading, PyCoMo's SBML format preserves the metabolite-ID prefixing (memberId_metaboliteId) and the shared exchange compartment (named 'medium' or set via a shared_compartment_id parameter), which downstream tools such as ScyNet/cy3sbml rely on to attribute metabolites to community members. The generated model keeps all metabolites, reactions, genes and compartments attributable to their member of origin and retains the original reaction bounds of the input member models, so round-tripping through SBML should not alter these.

## Related tools

- **PyCoMo** (Generates and saves/loads the community metabolic models in SBML format) — http://github.com/univieCUBE/PyCoMo
- **COBRApy** (Reloads the saved community SBML model and applies further COBRA methods directly)
- **SBML** (Serialization format used to save and reuse PyCoMo community models)
- **openCOBRA** (File-format compliance target for the generated community models)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('community_model.sbml')
```

## Evaluation signals

- community_model.sbml reloads as a COBRApy model without errors (cobra.io.read_sbml_model).
- All metabolites, reactions, genes and compartments remain attributable to their member of origin after the SBML round-trip.
- The original reaction bounds of the three input member models are preserved in the reloaded model.
- SBML file contains a shared exchange compartment named 'medium' (or a shared_compartment_id parameter) with member-prefixed metabolite IDs (memberId_metaboliteId).
- The model remains compliant with current openCOBRA file formats.

## Limitations

- The SBML format conventions are specific to PyCoMo-generated models; arbitrary SBML models without the shared exchange compartment and member-ID prefixing will not be correctly attributed by downstream tools such as ScyNet.
- Loading large community models into cy3sbml-based viewers can take a couple of minutes depending on hardware (>10 members).

## Evidence

- [methods] SBML save/load capability: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] COBRApy compatibility of community models: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] Member attribution and bound retention: "As also all metabolites, reactions, genes and compartments are directly attributable to their member of origin, PyCoMo community metabolic models are fully reusable."
- [readme] ScyNet SBML format requirements: [evidence span withheld — anchor status: external_doc]
