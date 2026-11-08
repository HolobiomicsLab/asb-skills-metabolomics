---
name: in-silico-class-prediction-interpretation
description: Use when a GNPS feature-based molecular networking job has been paired
  with CANOPUS class predictions (canopus_summary.tsv) and you need to propagate consistent
  chemical class annotations to unannotated ion features in subnetworks.
when_to_use_negative:
- When you only have raw spectra without GNPS molecular networking results and SIRIUS/CANOPUS
  class predictions — the consensus strategy requires both library matches and in
  silico class annotations.
- When your analysis relies on NPClassifier ontology output and you need ClassyFire
  ChemOnt terminology (or vice versa) without confirming the ontology argument, since
  mixed ontologies are not supported in a single run.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ConCISE
  role: CLI/GUI tool that fuses molecular networking, spectral library matching, and
    in silico class predictions into consensus ChemOnt annotations for subnetworks
  repo: https://github.com/Zquinlan/conCISE
- name: SIRIUS
  role: Generates in silico structural annotations feeding CANOPUS class predictions
- name: CANOPUS
  role: Produces the canopus_summary.tsv in silico class predictions used as consensus
    input
- name: GNPS
  role: Provides feature-based molecular networking task IDs, Node_info.tsv, and spectral
    library matches with ClassyFire ontologies
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: ClassyFire
  role: Supplies the ChemOnt hierarchical ontology (superclass, class, subclass) used
    for consensus classification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:8305919e365bfe5acc21511f96adf9260b04076affc6633ba1e5b297e12bc87b
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
yes_rate: 0.9285714285714286
n_claims: 14
---

# In silico class prediction interpretation

## Summary

Interprets SIRIUS/CANOPUS in silico structural class predictions by fusing them with GNPS spectral library matches and molecular networking to assign consensus ClassyFire (ChemOnt) annotations to entire molecular subnetworks.

## When to use

Use when a GNPS feature-based molecular networking job has been paired with CANOPUS class predictions (canopus_summary.tsv) and you need to propagate consistent chemical class annotations to unannotated ion features in subnetworks.

## When NOT to use

- When you only have raw spectra without GNPS molecular networking results and SIRIUS/CANOPUS class predictions — the consensus strategy requires both library matches and in silico class annotations.
- When your analysis relies on NPClassifier ontology output and you need ClassyFire ChemOnt terminology (or vice versa) without confirming the ontology argument, since mixed ontologies are not supported in a single run.

## Inputs

- GNPS feature-based molecular networking task ID (or spectral library match / DBResult file with superclass, class, subclass columns)
- CANOPUS in silico class prediction summary file (canopus_summary.tsv) generated via SIRIUS/CANOPUS
- GNPS node information file (Node_info.tsv)

## Outputs

- Consensus annotation file for each dataset (exported to a working directory)
- Per-dataset consensus annotation rates (e.g., 58.6%, 56.3%, 76.5%) and fold-expansion of annotation coverage over spectral library matching alone

## How to apply

Run ConCISE (python3 src/conciseCLI.py) supplying the GNPS task ID or spectral library match file, the CANOPUS canopus_summary.tsv, and the GNPS Node_info.tsv file. Set Superclass percent consensus = 50, Class percent consensus = 70, and Subclass percent consensus = 70, and use ClassyFire ChemOnt ontologies (NPClassifier = False) so that library matches and in silico predictions share the same ontology. ConCISE propagates an annotation to a subnetwork only when the majority of ion features share the identical structural class, exploiting the hierarchical consensus strategy across superclass, class, and subclass levels. Export the consensus annotation file per dataset, count annotated ion features over total ion features to compute the consensus annotation rate, and compute fold expansion over GNPS spectral library matching alone (reported up to 1244% across three case studies with rates of 58.6%, 56.3%, and 76.5%).

## Related tools

- **ConCISE** (CLI/GUI tool that fuses molecular networking, spectral library matching, and in silico class predictions into consensus ChemOnt annotations for subnetworks) — https://github.com/Zquinlan/conCISE
- **SIRIUS** (Generates in silico structural annotations feeding CANOPUS class predictions)
- **CANOPUS** (Produces the canopus_summary.tsv in silico class predictions used as consensus input)
- **GNPS** (Provides feature-based molecular networking task IDs, Node_info.tsv, and spectral library matches with ClassyFire ontologies) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **ClassyFire** (Supplies the ChemOnt hierarchical ontology (superclass, class, subclass) used for consensus classification)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Consensus annotation rates per dataset fall near the published case-study values (58.6%, 56.3%, 76.5%) when using the same thresholds (50/70/70) and ClassyFire ontologies.
- Annotation coverage expansion over GNPS spectral library matching alone should be positive and can reach up to 1244%.
- True positive rate of propagated consensus annotations should exceed 95% across superclass, class, and subclass levels of the ChemOnt ontology.
- Output consensus file contains entries only for ion features whose subnetwork majority shares identical structural classes at the required consensus thresholds.
- The consensus file is written to the specified export directory (default: current working directory) with columns matching the ChemOnt levels.

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; workarounds are switching to NPClassifier (NPC argument True) or manually retrieving ClassyFire data via the Fiehn Labs classyfire Batch service using InChIKeys from the GNPS DBResult file and merging them back with correct column names (superclass, class, subclass).
- The Mac GUI is deprecated; Mac users must use the CLI or MyBinder interface.
- Consensus propagation is intentionally conservative — annotations are only propagated when identical structural classes are shared by the majority of ion features in a subnetwork, so coverage gain depends on subnetwork coherence.

## Evidence

- [methods] Consensus thresholds and CLI usage: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False"
- [methods] Consensus mechanism: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [abstract] Propagation criterion: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [abstract] Performance benchmark: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [readme] GNPS ClassyFire workaround: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
- [readme] CLI arguments: "Superclass percent consensus (optional; default = 50)"
- [abstract] Coverage gain: "The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%"
