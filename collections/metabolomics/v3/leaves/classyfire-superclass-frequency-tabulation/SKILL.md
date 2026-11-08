---
name: classyfire-superclass-frequency-tabulation
description: Decision support for choosing between CANOPUS and MolNetEnhancer before
  running chemical class matching (e.g. NPClassScore), or for auditing how much superclass
  predictions disagree when both tools annotate the same GNPS molecular networking
  spectra.
when_to_use_negative:
- When the goal is BGC–MS/MS linking itself rather than auditing annotation-tool consistency
  — use the full NPLinker pipeline with NPClassScore instead.
- When the spectra under study are larger than the ~850 Da mass cut-off that limits
  CANOPUS coverage, since CANOPUS predictions may be absent and the comparison becomes
  biased.
- When only one annotation tool's predictions exist for the spectra — there is no
  second tool to compare against, so the tabulation is meaningless.
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CANOPUS
  role: Produces structure-based ClassyFire superclass predictions directly from MS/MS
    spectra via SIRIUS, run on the GNPS mgf file within NPLinker; one arm of the frequency
    comparison.
  repo: github.com/kaibioinfo/canopus_treemap
- name: MolNetEnhancer
  role: Provides the alternative ClassyFire superclass annotations on the GNPS platform;
    must be run externally since it cannot be executed within NPLinker.
- name: NPLinker
  role: Platform that runs CANOPUS on the GNPS mgf file and consumes the resulting
    class predictions; also hosts NPClassScore which is downstream of these superclass
    predictions.
  repo: github.com/NPLinker/nplinker
- name: ClassyFire
  role: The hierarchical ontology (superclass, class, subclass and seven more detailed
    levels) whose superclass category frequencies are tabulated for each tool.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:69ebeaa7237ef874012e841cf4cfabc14dce32b196ece625e0bb004987b7ca9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ClassyFire
  - MolNetEnhancer
  - NPLinker
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# classyfire-superclass-frequency-tabulation

## Summary

Tabulates the frequency of each ClassyFire superclass category predicted by CANOPUS and by MolNetEnhancer for the same set of MS/MS spectra, exposing systematic discrepancies between the two structure-based ontology prediction tools. Used to quantify how strongly the choice of annotation tool (CANOPUS via SIRIUS on GNPS mgf files vs MolNetEnhancer run externally on GNPS) influences downstream chemical class matching outcomes such as NPClassScore.

## When to use

Decision support for choosing between CANOPUS and MolNetEnhancer before running chemical class matching (e.g. NPClassScore), or for auditing how much superclass predictions disagree when both tools annotate the same GNPS molecular networking spectra.

## When NOT to use

- When the goal is BGC–MS/MS linking itself rather than auditing annotation-tool consistency — use the full NPLinker pipeline with NPClassScore instead.
- When the spectra under study are larger than the ~850 Da mass cut-off that limits CANOPUS coverage, since CANOPUS predictions may be absent and the comparison becomes biased.
- When only one annotation tool's predictions exist for the spectra — there is no second tool to compare against, so the tabulation is meaningless.

## Inputs

- CANOPUS superclass prediction output for GNPS molecular networking spectra (generated via SIRIUS v4.9.3 on the GNPS mgf file within NPLinker)
- MolNetEnhancer superclass annotations for the same spectra (run externally, e.g. on the GNPS platform)
- Set of MS/MS spectra from a paired-omics dataset (e.g. Streptomyces/Salinispora dataset from Crüsemann et al., MassIVE accessions MSV000078836, MSV000078839, MSV000079284)

## Outputs

- Per-ClassyFire-superclass frequency counts for CANOPUS and MolNetEnhancer separately over the shared spectra
- Discrepancy/agreement statistics per superclass category (e.g. 2686 vs 1301 for 'Lipids and lipid-like molecules')
- Structured comparison table of superclass counts suitable for validating against Tables S6–S7

## How to apply

Load the CANOPUS superclass predictions produced by running SIRIUS v4.9.3/CANOPUS directly on the GNPS molecular networking mgf file within NPLinker, together with the MolNetEnhancer superclass annotations generated externally on the GNPS platform. Filter to only those spectra for which both tools returned a ClassyFire superclass prediction — for the Streptomyces/Salinispora dataset (Crüsemann et al.) this yields 6606 spectra, excluding the 1988 spectra lacking a prediction from either tool. For each ClassyFire superclass category, count how many spectra each tool assigns to it (e.g. 'Lipids and lipid-like molecules': 2686 by MolNetEnhancer vs 1301 by CANOPUS), then compute per-category agreement/discrepancy metrics. Assemble the counts into a structured comparison table that can be validated against the article's Tables S6–S7 and used to decide which tool's predictions to feed into NPClassScore filtering (cut-off 0.25 after standardised Metcalf scoring at 2.5).

## Related tools

- **CANOPUS** (Produces structure-based ClassyFire superclass predictions directly from MS/MS spectra via SIRIUS, run on the GNPS mgf file within NPLinker; one arm of the frequency comparison.) — github.com/kaibioinfo/canopus_treemap
- **MolNetEnhancer** (Provides the alternative ClassyFire superclass annotations on the GNPS platform; must be run externally since it cannot be executed within NPLinker.)
- **NPLinker** (Platform that runs CANOPUS on the GNPS mgf file and consumes the resulting class predictions; also hosts NPClassScore which is downstream of these superclass predictions.) — github.com/NPLinker/nplinker
- **ClassyFire** (The hierarchical ontology (superclass, class, subclass and seven more detailed levels) whose superclass category frequencies are tabulated for each tool.)

## Examples

```
npl.load_data()  # then: link_graph = npl.get_links(npl.gcfs[:3], "metcalf")  # compare CANOPUS vs MolNetEnhancer superclass counts per spectrum
```

## Evaluation signals

- Reproducing the 2686-vs-1301 count for 'Lipids and lipid-like molecules' from Tables S6–S7 on the Streptomyces/Salinispora dataset.
- Confirming the shared-spectrum total equals 6606 (i.e. 6606 spectra have predictions from both tools, with 1988 excluded for lacking a prediction from either).
- Verifying that every superclass category's counts sum consistently across tools and that no category appears for one tool but not the other without an explanation.
- Checking that the per-category discrepancy magnitudes (e.g. for 'Organic acids and derivatives' and 'Phenylpropanoids and polyketides') are large enough to demonstrate the reported 'stark differences'.
- Confirming that the resulting table, when fed forward, correctly motivates the NPClassScore filtering behaviour (e.g. discarding 1988 spectra without structure-based classes).

## Limitations

- CANOPUS coverage is limited by an 850 Da mass cut-off; larger metabolites are not well handled by current tools and may be absent from the comparison.
- 1988 MS/MS spectra across the datasets lack a CANOPUS or MolNetEnhancer prediction, so candidate links for those spectra are discarded if filtering is enabled — the tabulation is based only on the remaining 6606 spectra.
- MolNetEnhancer cannot currently be run within NPLinker and must be executed externally (e.g. on the GNPS platform), complicating reproducible pipelines.
- Discrepancy counts alone do not resolve which tool is 'correct'; they only demonstrate that tool choice strongly influences NPClassScore outcomes.

## Evidence

- [discussion] Lipid superclass discrepancy counts: [evidence span withheld — anchor status: not_found]
- [discussion] Spectra with both tools' predictions: [evidence span withheld — anchor status: not_found]
- [discussion] Spectra lacking predictions: [evidence span withheld — anchor status: not_found]
- [discussion] CANOPUS run method: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [discussion] MolNetEnhancer run method: "Currently, MolNetEnhancer has to be run externally, for example on the GNPS platform"
- [intro] ClassyFire ontology hierarchy: "One of these is ClassyFire: a hierarchical ontology consisting of superclass, class, and subclass categories and seven more detailed levels"
- [discussion] Impact on NPClassScore: [evidence span withheld — anchor status: not_found]
