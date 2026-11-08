---
name: spectra-chemical-class-annotation-comparison
description: Use when two or more chemical-class annotation tools (e.g., CANOPUS vs
  MolNetEnhancer) have been run on the same GNPS molecular networking spectra and
  you need to quantify per-ontology-category agreement or discrepancy before choosing
  class-based link filtering thresholds.
when_to_use_negative:
- When only one structure-based annotation tool is available (no CANOPUS or no MolNetEnhancer
  output), as the intersection-based comparison cannot be performed and links must
  instead be discarded via filter_missing_scores
- When spectra are above the ~850 Da mass cut-off that limits CANOPUS coverage, since
  larger metabolites are not well handled by current tools and annotations will be
  sparse
- When the goal is purely co-occurrence-based (Metcalf) scoring without any class-based
  filtering — the comparison only informs NPClassScore application
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: CANOPUS
  role: Produces structure-based ClassyFire superclass predictions directly from MS/MS
    spectra via Sirius v4.9.3 on the GNPS mgf file within NPLinker
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: MolNetEnhancer
  role: Produces structure-based ClassyFire superclass annotations externally on the
    GNPS platform for the same spectra
- name: NPLinker
  role: Platform that runs CANOPUS on mgf files and applies NPClassScore filtering
    informed by the class-discrepancy analysis
  repo: https://github.com/NPLinker/nplinker
- name: Sirius
  role: Computational backbone invoked to run CANOPUS directly on the mgf file resulting
    from GNPS molecular networking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:aa0e989a098b147440b51eedbbadaf936d5f9b622312a9efad50b97b08eb2473
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kaibioinfo/canopus_treemap
  tools:
  - CANOPUS
  - MolNetEnhancer
  - NPLinker
  - Sirius
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectra-chemical-class-annotation-comparison

## Summary

Systematically compares ClassyFire superclass predictions from two structure-based tools, CANOPUS (run via Sirius on GNPS mgf files) and MolNetEnhancer (run externally on the GNPS platform), across the same MS/MS spectra, quantifying per-category disagreements that drive NPClassScore link ranking.

## When to use

Use when two or more chemical-class annotation tools (e.g., CANOPUS vs MolNetEnhancer) have been run on the same GNPS molecular networking spectra and you need to quantify per-ontology-category agreement or discrepancy before choosing class-based link filtering thresholds.

## When NOT to use

- When only one structure-based annotation tool is available (no CANOPUS or no MolNetEnhancer output), as the intersection-based comparison cannot be performed and links must instead be discarded via filter_missing_scores
- When spectra are above the ~850 Da mass cut-off that limits CANOPUS coverage, since larger metabolites are not well handled by current tools and annotations will be sparse
- When the goal is purely co-occurrence-based (Metcalf) scoring without any class-based filtering — the comparison only informs NPClassScore application

## Inputs

- CANOPUS superclass prediction output for GNPS mgf spectra (produced via Sirius v4.9.3 inside NPLinker)
- MolNetEnhancer superclass annotations obtained externally on the GNPS platform
- GNPS molecular networking mgf file for the Streptomyces/Salinispora (Crüsemann et al.) dataset
- Mapping of spectra IDs to filenames (e.g., extract_mappings_ms_filename_spectrum_id output)

## Outputs

- Structured comparison table of ClassyFire superclass counts per tool (CANOPUS vs MolNetEnhancer)
- Per-category discrepancy statistics (e.g., 2686 vs 1301 for 'Lipids and lipid-like molecules')
- Filtered set of spectra with predictions from both tools (6606 in the case study) and excluded spectra count (1988)
- Agreement metrics table suitable for informing NPClassScore cut-off decisions

## How to apply

Load the ClassyFire superclass predictions produced by CANOPUS (via Sirius v4.9.3 on the GNPS mgf file within NPLinker) and by MolNetEnhancer (run externally on the GNPS platform) for the paired-omics spectra. Intersect the two annotation sets to retain only spectra carrying a superclass prediction from both tools — in the Streptomyces/Salinispora dataset this yields 6606 comparable spectra out of 8594 total (1988 lack a prediction from one tool). Tabulate ClassyFire superclass frequencies independently for each tool and compute per-category discrepancy statistics, highlighting categories where the tools diverge most, such as 'Lipids and lipid-like molecules' (MolNetEnhancer 2686 vs CANOPUS 1301). Assemble a structured comparison table of superclass counts and agreement metrics, then use the identified discrepancies to justify the filter_missing_scores toggle that discards candidate BGC-MS/MS links without structure-based classes.

## Related tools

- **CANOPUS** (Produces structure-based ClassyFire superclass predictions directly from MS/MS spectra via Sirius v4.9.3 on the GNPS mgf file within NPLinker) — https://github.com/kaibioinfo/canopus_treemap
- **MolNetEnhancer** (Produces structure-based ClassyFire superclass annotations externally on the GNPS platform for the same spectra)
- **NPLinker** (Platform that runs CANOPUS on mgf files and applies NPClassScore filtering informed by the class-discrepancy analysis) — https://github.com/NPLinker/nplinker
- **Sirius** (Computational backbone invoked to run CANOPUS directly on the mgf file resulting from GNPS molecular networking)

## Examples

```
from nplinker import NPLinker; npl = NPLinker(); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], 'metcalf')
```

## Evaluation signals

- Total spectra with predictions from both tools must equal 6606 for the Streptomyces/Salinispora dataset (excluding 1988 spectra lacking a prediction from either tool)
- The comparison table must reproduce the reported counts from Tables S6–S7, including 'Lipids and lipid-like molecules' predicted 2686 times by MolNetEnhancer versus 1301 times by CANOPUS
- Each ClassyFire superclass category must appear as an independent row with two count columns (one per tool), and no category should be missing from either side of the intersection
- The excluded-spectra count (1988) should match the sum of spectra in the mgf file that lack a CANOPUS or MolNetEnhancer prediction across all paired-omics datasets

## Limitations

- 1988 MS/MS spectra across the datasets lack a CANOPUS or MolNetEnhancer prediction, so any class-based comparison excludes them; discarding these links is implemented via the filter_missing_scores toggle
- MolNetEnhancer must be run externally (e.g., on the GNPS platform) as it cannot currently be run within NPLinker, requiring separate pipeline integration
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are not well handled by current tools
- Merged antiSMASH 3 and antiSMASH 6 outputs introduce version inconsistency, though this affects the genomic side rather than the class annotation comparison itself

## Evidence

- [results] MolNetEnhancer predicts 2686 occurrences versus 1301 for CANOPUS: [evidence span withheld — anchor status: not_found]
- [results] 6606 spectra with predictions from both tools: [evidence span withheld — anchor status: not_found]
- [discussion] CANOPUS run via Sirius on mgf within NPLinker: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [discussion] MolNetEnhancer must be run externally: "Currently, MolNetEnhancer has to be run externally, for example on the GNPS platform"
- [discussion] filter_missing_scores discards links without structure-based classes: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [intro] CANOPUS and MolNetEnhancer are the two structure-based prediction tools: "predicting these structure-based ontologies directly from MS/MS spectra has also become possible through CANOPUS and MolNetEnhancer"
