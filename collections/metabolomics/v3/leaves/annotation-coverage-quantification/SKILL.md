---
name: annotation-coverage-quantification
description: Use when you have run ConCISE consensus annotation propagation on GNPS
  feature-based molecular networking jobs and need to report how many ion features
  were annotated and how much coverage expanded beyond library matches.
when_to_use_negative:
- The goal is to evaluate annotation accuracy (true positive rate) rather than coverage;
  use the ConCISE validation workflow instead
- Inputs are raw MS/MS spectra that have not been processed by GNPS feature-based
  molecular networking and SIRIUS/CANOPUS, so no putative annotations exist to tally
- The consensus run used NPClassifier ontologies, which is incompatible with the ClassyFire
  ChemOnt-based rate comparison against ClassyFire-supplied library matches
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Runs the consensus annotation propagation whose output files are tallied to
    compute coverage rates
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Supplies feature-based molecular networking jobs, Node_info.tsv, and spectral
    library match annotations used as the coverage baseline
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: CANOPUS
  role: Provides in silico class predictions (canopus_summary.tsv) feeding the consensus
    annotations being counted
  repo: https://bio.informatik.uni-jena.de/software/
- name: Python
  role: Runs the ConCISE CLI (src/conciseCLI.py) and the counting/division computation
    for annotation rates
- name: ClassyFire
  role: Defines the ChemOnt superclass/class/subclass ontology levels across which
    annotation coverage is quantified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:35618cdb4f3cbe3f86e3c35b893dc34d1fd304accecbe9877841b533f7ce3a6b
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# annotation-coverage-quantification

## Summary

Quantifies the proportion of ion features receiving a ConCISE consensus annotation per dataset and computes the fold expansion of annotation coverage relative to GNPS spectral library matching alone, e.g. rates of 58.6%, 56.3%, and 76.5% across three case studies.

## When to use

Use when you have run ConCISE consensus annotation propagation on GNPS feature-based molecular networking jobs and need to report how many ion features were annotated and how much coverage expanded beyond library matches.

## When NOT to use

- The goal is to evaluate annotation accuracy (true positive rate) rather than coverage; use the ConCISE validation workflow instead
- Inputs are raw MS/MS spectra that have not been processed by GNPS feature-based molecular networking and SIRIUS/CANOPUS, so no putative annotations exist to tally
- The consensus run used NPClassifier ontologies, which is incompatible with the ClassyFire ChemOnt-based rate comparison against ClassyFire-supplied library matches

## Inputs

- GNPS feature-based molecular networking task ID or spectral library match file (DBResult with superclass/class/subclass columns)
- CANOPUS in silico class prediction summary file (canopus_summary.tsv) from SIRIUS/CANOPUS
- GNPS Node_info.tsv networking information file per dataset

## Outputs

- Consensus annotation file per dataset (exported to working directory)
- Per-dataset consensus annotation rate (e.g. 58.6%, 56.3%, 76.5%)
- Fold expansion of annotation coverage relative to GNPS spectral library matching alone (up to 1244%)
- Summary table (annotation_rates.csv)

## How to apply

Retrieve the GNPS feature-based molecular networking job identifiers and Node_info.tsv files for each dataset, plus the CANOPUS in silico class prediction summary (canopus_summary.tsv) generated via SIRIUS/CANOPUS. Run ConCISE via its CLI with Superclass percent consensus = 50, Class percent consensus = 70, Subclass percent consensus = 70, and NPClassifier = False, since ClassyFire ChemOnt ontologies are used for both library matches and in silico annotations. Export the consensus annotation file per dataset to the working directory, count ion features receiving a consensus annotation, and divide by the total ion features per dataset to obtain the annotation rate. Finally compute the fold expansion of coverage relative to GNPS spectral library matching alone (up to 1244% in the case studies) and assemble the per-dataset summary table (e.g. annotation_rates.csv).

## Related tools

- **ConCISE** (Runs the consensus annotation propagation whose output files are tallied to compute coverage rates) — https://github.com/Zquinlan/conCISE
- **GNPS** (Supplies feature-based molecular networking jobs, Node_info.tsv, and spectral library match annotations used as the coverage baseline) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **CANOPUS** (Provides in silico class predictions (canopus_summary.tsv) feeding the consensus annotations being counted) — https://bio.informatik.uni-jena.de/software/
- **Python** (Runs the ConCISE CLI (src/conciseCLI.py) and the counting/division computation for annotation rates)
- **ClassyFire** (Defines the ChemOnt superclass/class/subclass ontology levels across which annotation coverage is quantified)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Per-dataset consensus annotation rate computed as annotated ion features divided by total ion features; case-study benchmarks are 58.6%, 56.3%, and 76.5%
- Fold expansion versus GNPS spectral library matching alone reaches up to 1244%; expansion values should be reproducible from the same consensus and DBResult files
- Consensus run used consensus thresholds 50/70/70 (superclass/class/subclass) and NPC=False; rates computed with different thresholds are not comparable to the published benchmarks
- Exported consensus annotation file exists in the specified working directory for each dataset before counting
- Underlying annotations must come from ClassyFire ChemOnt ontology levels; GNPS no longer supplies ClassyFire ontology information for library matches, so the library baseline must be reconstructed

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches, so the library-match baseline requires a workaround (NPClassifier ontologies, or manually pulling ClassyFire data for GNPS DBResult InChiKeys via the Fiehn Labs classyfire Batch service and merging with correct column names)
- Coverage expansion (up to 1244%) depends on consensus thresholds (Superclass 50%, Class 70%, Subclass 70%); changing thresholds changes the rate
- Only ion features within molecular networking subnetworks propagate annotations; coverage applies to features represented in the GNPS network and canopus_summary inputs

## Evidence

- [results] ConCISE annotated 58.6%, 56.3%, and 76.5% of ion features in the three datasets, expanding subnetwork annotation rate by up to 1244% over spectral library matches alone.: [evidence span withheld — anchor status: not_found]
- [abstract] The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%: "The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%"
- [methods] python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv: "python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv"
- [readme] Currently GNPS has stopped supplying classyfire ontology information for spectral library matches.: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
