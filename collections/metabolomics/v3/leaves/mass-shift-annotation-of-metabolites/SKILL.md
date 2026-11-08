---
name: mass-shift-annotation-of-metabolites
description: Use when a paired transcriptomic-metabolomic dataset contains unannotated
  mass features whose m/z differences (mass shifts) must be assigned putative reaction
  identities and enzyme families, without prior 'bait' knowledge.
when_to_use_negative:
- The candidate reactions involve two substrates (e.g. p-coumaroyltyramine from tyrosine
  plus p-coumaroyl-CoA), since RetroRules-based rules are designed for single-substrate
  reactions only.
- The analysis requires stereochemical resolution or MS/MS spectral library annotation,
  which MEANtools does not currently support and mass spectrometric data do not capture.
- There is no paired transcriptomic data to establish correlated enzyme-metabolite
  sets, since the filtering of mass shift-reaction associations depends on those correlations.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MEANtools
  role: Executes the MR-based correlation, adduct-based structure prediction against
    LOTUS, and mass shift-reaction association filtering
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules and reaction-enzyme associations (loose/medium/strict)
    used to assign reaction identities to mass shifts
- name: LOTUS
  role: Metabolite structure database queried (in SQLite format) to assign molecular
    structures to mass features after adduct identification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:1bd32e02edea1050f0198deff75e093810b6bf49cbabf03d4154f51240271287
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-shift annotation of metabolites

## Summary

MEANtools annotates untargeted metabolomics mass features with predicted structures (adduct detection + LOTUS lookup at 20 ppm tolerance) and links mass shifts between correlated metabolite pairs to RetroRules reaction rules, filtering associations by cross-checking reaction-enzyme datasets against correlated enzyme-coding genes and metabolites.

## When to use

Use when a paired transcriptomic-metabolomic dataset contains unannotated mass features whose m/z differences (mass shifts) must be assigned putative reaction identities and enzyme families, without prior 'bait' knowledge.

## When NOT to use

- The candidate reactions involve two substrates (e.g. p-coumaroyltyramine from tyrosine plus p-coumaroyl-CoA), since RetroRules-based rules are designed for single-substrate reactions only.
- The analysis requires stereochemical resolution or MS/MS spectral library annotation, which MEANtools does not currently support and mass spectrometric data do not capture.
- There is no paired transcriptomic data to establish correlated enzyme-metabolite sets, since the filtering of mass shift-reaction associations depends on those correlations.

## Inputs

- paired transcriptomic-metabolomic dataset (e.g. the tomato Jeon and colleagues 2020 dataset) with mass feature m/z values and gene expression counts
- RetroRules-derived reaction rules dataset (loose, medium, and strict entries combined, e.g. Combined_small_middle_big_datasets.csv from Zenodo record 15697913)
- LOTUS metabolite structure database converted to SQLite format
- mass feature list with detected adducts at 20 ppm mass accuracy tolerance

## Outputs

- predicted metabolite structures per mass feature (adduct-based annotation from LOTUS)
- filtered set of mass shift-reaction associations linking correlated enzyme-coding genes and metabolites
- directed reaction network (reaction_network.csv) whose edges are annotated mass shifts with associated reaction rules and Pfam families

## How to apply

Annotate each mass feature by detecting possible adducts and querying the LOTUS database (SQLite-converted) or a user-defined metabolite database, operating at the default 20 ppm mass accuracy tolerance to balance sensitivity and specificity. Convert pairwise transcript-mass feature Pearson correlations to mutual ranks (MR) and exponential-decay edge weights, excluding gene-metabolite pairs with edge scores below 0.01, so that correlated enzyme-coding genes and metabolite sets are available. Cross-check the RetroRules-derived reaction-enzyme association datasets (loose, medium, strict entries combined) with these correlated sets to filter the candidate mass shift-reaction associations, preferring the strict (experimentally validated) ruleset to avoid spurious Pfam links retained in the loose set. Mass shifts between metabolite pairs that match a reaction rule become edges in the directed reaction network, with unmeasured intermediates kept as ghost mass signatures. Filter rules involving stereomeric structures to avoid complexity, since mass spectrometric data do not capture stereoisomers.

## Related tools

- **MEANtools** (Executes the MR-based correlation, adduct-based structure prediction against LOTUS, and mass shift-reaction association filtering) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules and reaction-enzyme associations (loose/medium/strict) used to assign reaction identities to mass shifts)
- **LOTUS** (Metabolite structure database queried (in SQLite format) to assign molecular structures to mass features after adduct identification)

## Evaluation signals

- Each annotated mass feature is matched within the 20 ppm mass accuracy tolerance to a LOTUS structure via an identified adduct.
- Every retained mass shift-reaction association is backed by at least one correlated enzyme-coding gene-metabolite pair with edge score >= 0.01 in the Ni→j networks.
- Using the strict ruleset, the annotated enzyme for a known conversion is correct (benchmark: octadecadiene-diynoic acid conversion to metabolite 6 as reported by Jeon and colleagues 2020).
- Annotated pathway steps can be quantitatively compared to a characterized pathway (benchmark: 5 of 7 falcarindiol steps anticipated).
- Spurious reaction-Pfam links from the loose dataset (e.g. AminoTran_1_2, Glyco_transf_20) are flagged or excluded by applying a Pfam cutoff of 6 and preferring strict entries.

## Limitations

- Loose RetroRules predictions can be associated with unrelated Pfams (e.g. AminoTran_1_2, Glyco_transf_20) after the Pfam cutoff of 6, so strict rules should be used when available.
- Stereomeric conversions (e.g. phenylalanine to p-coumaroyl-CoA) are filtered because mass spectrometric data do not capture stereoisomers, hindering reconstruction of steps involving them.
- Mass spectrometer resolution directly impacts the precision of detected m/z values and the confidence of adduct-based matching at the 20 ppm tolerance.
- Annotating mass signatures with predicted structures could be improved by loading MS/MS data and using spectral library and networking-based annotation, which is not currently supported.

## Evidence

- [discussion] filters possible mass shift-reaction associations: "By cross-checking these reaction-enzyme association datasets with sets of correlated enzyme-coding genes and metabolites, MEANtools effectively filters the set of possible mass shift-reaction"
- [discussion] adduct-based structure prediction: "it predicts possible metabolites and their corresponding molecular structures for each mass feature by identifying possible adducts and querying the LOTUS database"
- [discussion] 20 ppm default tolerance: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [discussion] LOTUS SQLite compatibility: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
