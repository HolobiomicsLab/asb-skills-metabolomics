---
name: chemical-class-based-link-filtering
description: Use when a genome–metabolome pairing workflow yields long candidate lists
  of GCF–spectrum links (e.g. from Metcalf scoring) and class-ontology annotations
  (CANOPUS/MolNetEnhancer for spectra, MIBiG-derived classes for BGCs) are available
  to prune unlikely matches.
when_to_use_negative:
- When spectra lack CANOPUS or MolNetEnhancer predictions and retaining unclassified
  links is required — discarding them changes the reported reductions (49% average
  decrease in candidate links per GCF if not filtered)
- When the goal is de novo linking without any chemical-class ontology predictions
  (e.g. pure co-occurrence analysis), since NPClassScore requires structure-based
  classes for both BGC and spectrum
- When working with metabolites above the ~850 Da CANOPUS mass cut-off, where class
  predictions are unreliable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Platform implementing both standardised Metcalf scoring and NPClassScore filtering
    of GCF–MS/MS links
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical class matching score applied as a cut-off (0.25) to filter candidate
    links
  repo: https://github.com/NPLinker/nplinker
- name: CANOPUS
  role: Predicts structure-based chemical classes from MS/MS spectra (Sirius v4.9.3
    on the GNPS mgf file)
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: MolNetEnhancer
  role: Alternative structure-based class annotation run externally on the GNPS platform
- name: MIBiG classifications
  role: Notebooks/scripts to derive BGC-side chemical classes and the class-matching
    scoring tables from MIBiG 2.0
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:08593da026593c5b30c17d96b84724257d9627077faeac5b97f7306e754645f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - CANOPUS
  - MIBiG classifications
  - MolNetEnhancer
  - NPClassScore
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.92
n_claims: 25
---

# chemical-class-based-link-filtering

## Summary

Filter candidate BGC–MS/MS spectrum links produced by co-occurrence scoring (e.g. standardised Metcalf) using NPClassScore, an empirical chemical-class matching score implemented in NPLinker, to discard links whose matched NP ontologies are inconsistent.

## When to use

Use when a genome–metabolome pairing workflow yields long candidate lists of GCF–spectrum links (e.g. from Metcalf scoring) and class-ontology annotations (CANOPUS/MolNetEnhancer for spectra, MIBiG-derived classes for BGCs) are available to prune unlikely matches.

## When NOT to use

- When spectra lack CANOPUS or MolNetEnhancer predictions and retaining unclassified links is required — discarding them changes the reported reductions (49% average decrease in candidate links per GCF if not filtered)
- When the goal is de novo linking without any chemical-class ontology predictions (e.g. pure co-occurrence analysis), since NPClassScore requires structure-based classes for both BGC and spectrum
- When working with metabolites above the ~850 Da CANOPUS mass cut-off, where class predictions are unreliable

## Inputs

- GNPS molecular networking output (mgf file, MassIVE accessions e.g. MSV000078836, MSV000078839, MSV000079284, MSV000084950, MSV000084771)
- antiSMASH/BiG-SCAPE GCF annotations (BiG-SCAPE cut-off 0.3)
- MIBiG 2.0 json entries with SMILES and (sub)classes
- CANOPUS or MolNetEnhancer structure-based class predictions for MS/MS spectra
- nplinker.toml configuration for the PoDP dataset (podp mode with PoDP id)

## Outputs

- Filtered set of candidate BGC–MS/MS spectrum links per GCF (NPClassScore >= 0.25)
- Per-GCF link-count histograms before vs after filtering
- Per-dataset percentage reduction in candidate links per GCF
- Ranked positions of experimentally validated links before and after filtering

## How to apply

First generate candidate links with standardised Metcalf co-occurrence scoring in NPLinker (npl.get_links(npl.gcfs, "metcalf")) using a cut-off of 2.5, lowered to 2 for small datasets like Nocardia where validated links score low. Then match genome-based BGC classes to structure-based compound classes obtained from CANOPUS (run via Sirius v4.9.3 on the GNPS mgf file) or MolNetEnhancer (run externally on the GNPS platform), with BGC classes derived from MIBiG 2.0 json entries (SMILES plus (sub)classes, ClassyFire/NPClassifier retrieved via the GNPS API). Discard candidate links lacking structure-based classes (the 1988 spectra without CANOPUS/MolNetEnhancer predictions) and retain only links with NPClassScore >= 0.25. Re-count links per GCF to quantify reduction; expect on average a 63% decrease in candidate links per GCF across the three PoDP datasets, and record ranked positions of any experimentally validated links before and after filtering.

## Related tools

- **NPLinker** (Platform implementing both standardised Metcalf scoring and NPClassScore filtering of GCF–MS/MS links) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Empirical class matching score applied as a cut-off (0.25) to filter candidate links) — https://github.com/NPLinker/nplinker
- **CANOPUS** (Predicts structure-based chemical classes from MS/MS spectra (Sirius v4.9.3 on the GNPS mgf file)) — https://github.com/kaibioinfo/canopus_treemap
- **MolNetEnhancer** (Alternative structure-based class annotation run externally on the GNPS platform)
- **MIBiG classifications** (Notebooks/scripts to derive BGC-side chemical classes and the class-matching scoring tables from MIBiG 2.0) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Per-dataset candidate-link reductions match the reported values: 68% (Streptomyces/Salinispora), 53% (Cyanobacteria, 27→13 links per GCF), 69% (Nocardia, 206→64), averaging 63%
- Streptomyces/Salinispora post-filtering histogram shows 92 GCFs with ≤10 candidate links and 270 GCFs with ≤25 links, versus 5 and 42 pre-filtering
- 96% of experimentally validated links across the three PoDP datasets are retained and prioritised after NPClassScore filtering
- All retained links carry structure-based classes on both BGC and spectrum sides; links with missing CANOPUS/MolNetEnhancer predictions are absent
- Nocardia validated links must still be present after filtering when the Metcalf threshold is lowered from 2.5 to 2

## Limitations

- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions and were discarded; not filtering these decreases the average candidate links per GCF by only 49%
- No MS/MS spectrum could be found for the validated compounds retimycin and nocardimicin
- MolNetEnhancer must be run externally (e.g. on the GNPS platform); it cannot run within NPLinker
- CANOPUS coverage is limited by an 850 Da mass cut-off; Spec2Vec/MS2DeepScore may suit bigger metabolites
- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring threshold lowering to 2

## Evidence

- [discussion] Metcalf cut-off: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] abstract reduction: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [discussion] missing classes: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [readme] README method: [evidence span withheld — anchor status: external_doc]
- [discussion] Nocardia threshold: "the Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, which is why we changed the standardised Metcalf threshold from 2.5 to 2"
