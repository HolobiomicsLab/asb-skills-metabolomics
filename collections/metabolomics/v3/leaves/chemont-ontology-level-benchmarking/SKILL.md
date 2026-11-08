---
name: chemont-ontology-level-benchmarking
description: Use when evaluating whether a consensus annotation workflow assigns correct
  chemical classes at superclass, class, and subclass levels of the ChemOnt ontology,
  e.g. validating ConCISE output against GNPS-reported identities on MassIVE datasets.
when_to_use_negative:
- GNPS has stopped supplying ClassyFire ontology information for spectral library
  matches, so a ClassyFire-based library benchmark requires the workaround (pull ClassyFire
  data via the Fiehn Labs classyfire Batch for the InChiKeys in the GNPS DBResult
  file) or switching the NPClassifier flag to True
- The dataset lacks CANOPUS/in silico class predictions, so only the library-match
  condition (not the 96.1% ± 4.96% in silico condition) can be benchmarked
- The goal is molecular networking or feature detection itself rather than benchmarking
  classification accuracy across ChemOnt hierarchy levels
tools:
- name: ConCISE
  role: Computes consensus annotations at configurable superclass/class/subclass consensus
    thresholds; provides the CLI used for the benchmark
  repo: https://github.com/Zquinlan/conCISE
- name: GNPS
  role: Feature-based molecular networking produces Node_info.tsv and spectral library
    match files used as benchmark inputs
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: SIRIUS
  role: Runs CANOPUS to generate the canopus_summary.tsv in silico class predictions
  repo: https://bio.informatik.uni-jena.de/software/
- name: CANOPUS
  role: Supplies the in silico ChemOnt class predictions benchmarked in the in silico
    condition
- name: ClassyFire
  role: Defines the ChemOnt hierarchy (superclass, class, subclass) against which
    true positive rates are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:881018b9f9c87ffb07842c6164ee6a9040cf7ca5eb1eaaec6a184efbe15bdc96
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# chemont-ontology-level-benchmarking

## Summary

Benchmarks chemical classification accuracy at each level of the ClassyFire ChemOnt hierarchy (superclass, class, subclass) by comparing ConCISE consensus annotations against GNPS-reported compound identities, yielding per-level true positive rates (library 98.88% ± 1.62%, in silico 96.1% ± 4.96%).

## When to use

Use when evaluating whether a consensus annotation workflow assigns correct chemical classes at superclass, class, and subclass levels of the ChemOnt ontology, e.g. validating ConCISE output against GNPS-reported identities on MassIVE datasets.

## When NOT to use

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches, so a ClassyFire-based library benchmark requires the workaround (pull ClassyFire data via the Fiehn Labs classyfire Batch for the InChiKeys in the GNPS DBResult file) or switching the NPClassifier flag to True
- The dataset lacks CANOPUS/in silico class predictions, so only the library-match condition (not the 96.1% ± 4.96% in silico condition) can be benchmarked
- The goal is molecular networking or feature detection itself rather than benchmarking classification accuracy across ChemOnt hierarchy levels

## Inputs

- MassIVE case-study datasets (MSV000082083, MSV000081731, MSV000083729)
- GNPS feature-based molecular networking info file (Node_info.tsv)
- GNPS spectral library match file (for library-match condition)
- SIRIUS 4 / CANOPUS class prediction summary file (canopus_summary.tsv with ClassyFire ChemOnt annotations)

## Outputs

- Per-level true positive rate table (tpr_summary_table.tsv) with one row per dataset per consensus-threshold condition and ontology level
- ConCISE consensus annotation files (consensus superclass, class, subclass assignments)

## How to apply

Process each dataset (e.g. MassIVE MSV000082083, MSV000081731, MSV000083729) with GNPS feature-based molecular networking to obtain Node_info.tsv and, for library-match runs, the GNPS spectral library match file. Generate the CANOPUS class prediction summary (canopus_summary.tsv) by running SIRIUS 4 with CANOPUS on each dataset. Run the ConCISE CLI once with the library match file and once without it, using the paper's consensus thresholds (superclass 50%, class 70%, subclass 70%) so that propagation is limited to structural classes identical for the majority of ion features in a subnetwork. For each consensus output, compare the consensus superclass, class, and subclass assignments against GNPS-reported compound identities to compute a true positive rate per ontology level. Assemble the rates into a table with one row per dataset per consensus-threshold condition and level (tpr_summary_table.tsv) and check them against the reported criterion of a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology.

## Related tools

- **ConCISE** (Computes consensus annotations at configurable superclass/class/subclass consensus thresholds; provides the CLI used for the benchmark) — https://github.com/Zquinlan/conCISE
- **GNPS** (Feature-based molecular networking produces Node_info.tsv and spectral library match files used as benchmark inputs) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **SIRIUS** (Runs CANOPUS to generate the canopus_summary.tsv in silico class predictions) — https://bio.informatik.uni-jena.de/software/
- **CANOPUS** (Supplies the in silico ChemOnt class predictions benchmarked in the in silico condition)
- **ClassyFire** (Defines the ChemOnt hierarchy (superclass, class, subclass) against which true positive rates are computed)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- True positive rate exceeds 95% at all three ChemOnt levels (superclass, class, subclass), matching the published criterion
- Library-match consensus TPR falls near the reported 98.88% ± 1.62%
- In silico consensus TPR falls near the reported 96.1% ± 4.96% across Superclass, Class, Subclass
- Consensus thresholds are set to the defaults 50/70/70 (superclass/class/subclass percent consensus) when reproducing the paper's conditions
- tpr_summary_table.tsv contains one row per dataset per consensus-threshold condition per ontology level

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches, so library-match benchmarks need the NPClassifier substitution or manual ClassyFire enrichment via the Fiehn Labs classyfire Batch merged back into the GNPS DBResult file with superclass/class/subclass column names
- The Mac GUI is deprecated; on macOS the benchmark must be run via the CLI or MyBinder interface
- The in silico condition has higher variance (± 4.96%) than the library condition (± 1.62%)

## Evidence

- [abstract] TPR criterion across all levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [methods] consensus threshold defaults: "Superclass percent consensus (optional; default = 50)"
- [methods] consensus mechanism: "By limiting annotation propagation to only structural classes which are identical for the majority of ion features within a subnetwork"
- [readme] GNPS ClassyFire supply issue: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."
