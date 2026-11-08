---
name: consensus-annotation-propagation
description: Use when a GNPS feature-based molecular networking job has both spectral
  library matches (DBResult/Node_info) and SIRIUS/CANOPUS canopus_summary.tsv predictions,
  and you need consensus chemical class annotations for ion features and whole subnetworks.
when_to_use_negative:
- You only have raw MS/MS spectra without a GNPS feature-based molecular networking
  job or Node_info.tsv — ConCISE requires networking and library-match context.
- You need fully resolved structures, not chemical-class consensus — ConCISE assigns
  putative ChemOnt classifications, not identified metabolites.
- GNPS library matches lack ClassyFire ontology columns and you cannot apply the NPClassifier
  or manual ClassyFire Batch workarounds (cfb.fiehnlab.ucdavis.edu).
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ConCISE
  role: 'Core tool: computes consensus ChemOnt annotations by fusing molecular networking,
    spectral library matching, and in silico class predictions; run via CLI, GUI,
    or myBinder notebook'
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Provides feature-based molecular networking results, spectral library matches,
    and (historically) ClassyFire ontology columns for library matches
- name: SIRIUS
  role: In silico structure elucidation upstream of CANOPUS class predictions consumed
    by ConCISE
- name: CANOPUS
  role: Generates the canopus_summary.tsv in silico class predictions that ConCISE
    fuses with library matches
- name: ClassyFire
  role: Supplies the ChemOnt hierarchical ontology (superclass/class/subclass) against
    which consensus is computed
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:6fd79155808d3b5d1f00f6c6b7d22749bbb123295d912397711fe5b862b03809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - Python
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# consensus-annotation-propagation

## Summary

ConCISE fuses GNPS molecular networking, spectral library matching, and SIRIUS/CANOPUS in silico class predictions to assign consensus ClassyFire ChemOnt annotations (superclass, class, subclass) to entire molecular subnetworks, propagating annotations only when a structural class is shared by the majority of ion features.

## When to use

Use when a GNPS feature-based molecular networking job has both spectral library matches (DBResult/Node_info) and SIRIUS/CANOPUS canopus_summary.tsv predictions, and you need consensus chemical class annotations for ion features and whole subnetworks.

## When NOT to use

- You only have raw MS/MS spectra without a GNPS feature-based molecular networking job or Node_info.tsv — ConCISE requires networking and library-match context.
- You need fully resolved structures, not chemical-class consensus — ConCISE assigns putative ChemOnt classifications, not identified metabolites.
- GNPS library matches lack ClassyFire ontology columns and you cannot apply the NPClassifier or manual ClassyFire Batch workarounds (cfb.fiehnlab.ucdavis.edu).

## Inputs

- GNPS feature-based molecular networking task ID (or GNPS spectral library match / DBResult file)
- CANOPUS in silico class prediction summary file (canopus_summary.tsv) from SIRIUS/CANOPUS
- GNPS Node_info.tsv networking info file
- Consensus thresholds: superclass=50, class=70, subclass=70 (percent)
- Ontology flag: NPClassifier True/False (False = ClassyFire ChemOnt)

## Outputs

- Consensus annotation file (ChemOnt superclass/class/subclass per ion feature, exported to a chosen directory)
- Per-subnetwork consensus scores (e.g., consensus_scores.csv)
- Annotation-rate summaries (fraction of ion features / subnetworks annotated; fold expansion over library matching alone)

## How to apply

Run the ConCISE CLI (python3 src/conciseCLI.py) supplying the GNPS task ID (or a spectral library match file), the CANOPUS canopus_summary.tsv, and the GNPS Node_info.tsv file, with superclass/class/subclass percent consensus thresholds set to 50/70/70 and NPClassifier set to False so ClassyFire ChemOnt ontologies are used for both library matches and in silico annotations. ConCISE finds consensus annotations of putative annotations across superclass, class, and subclass levels, propagating a class to a subnetwork only when that class is identical for the majority of ion features within it. Export the consensus annotation file to a working directory, then count annotated ion features (or subnetworks) over the total to compute annotation rates and per-subnetwork consensus scores. Verify against published benchmarks such as 58.6%, 56.3%, and 76.5% ion-feature annotation rates and a >95% true positive rate across ChemOnt levels.

## Related tools

- **ConCISE** (Core tool: computes consensus ChemOnt annotations by fusing molecular networking, spectral library matching, and in silico class predictions; run via CLI, GUI, or myBinder notebook) — https://github.com/Zquinlan/conCISE
- **GNPS** (Provides feature-based molecular networking results, spectral library matches, and (historically) ClassyFire ontology columns for library matches)
- **SIRIUS** (In silico structure elucidation upstream of CANOPUS class predictions consumed by ConCISE)
- **CANOPUS** (Generates the canopus_summary.tsv in silico class predictions that ConCISE fuses with library matches)
- **ClassyFire** (Supplies the ChemOnt hierarchical ontology (superclass/class/subclass) against which consensus is computed)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Published benchmark reproduction: consensus annotation rates of 58.6%, 56.3%, and 76.5% of ion features across the three case-study datasets.
- True positive rate >95% across all ChemOnt levels (superclass, class, subclass) on validated data.
- Case Study 1 (MSV000082083, coral reef dissolved organic matter): 61.09% of molecular subnetworks annotated with mean consensus 87.4%; subnetwork 131 (43 annotated features) at 95.3% and fatty amide subnetwork 2263 at 72.0%.
- Subnetwork consensus never exceeds the majority threshold — a class is propagated only if identical for the majority of features in the subnetwork (superclass ≥50%, class/subclass ≥70% by default).
- Output file contains non-empty superclass, class, and subclass columns for the annotated ion features, and expansion over library matching alone (up to 1244%).

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC argument True) or manually pulling ClassyFire annotations via the Fiehn Labs ClassyFire Batch using InChiKeys from the GNPS DBResult file and merging them back with superclass/class/subclass column names.
- Mac GUI is deprecated; use CLI or myBinder on macOS.
- Annotation propagation is conservative by design — annotations are only propagated to subnetworks where the majority of ion features share the same structural class, so sparse subnetworks may remain unannotated.

## Evidence

- [abstract] ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology (superclass, class, subclass): "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [abstract] By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [methods] ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [methods] python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv"
- [readme] Currently GNPS has stopped supplying classyfire ontology information for spectral library matches: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [readme] Superclass percent consensus (optional; default = 50): "Superclass percent consensus (optional; default = 50)"
