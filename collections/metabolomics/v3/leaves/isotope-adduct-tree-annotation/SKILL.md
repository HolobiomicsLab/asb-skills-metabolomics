---
name: isotope-adduct-tree-annotation
description: Use when you have an asari- or other-preprocessed LC-MS feature table
  (tab-delimited feature ID, m/z, rtime, intensities) in positive or negative ESI
  mode and need to group degenerate ions into empirical compounds, including stable
  isotope tracing (e.g. U-13C-labeled E.
when_to_use_negative:
- When complete annotation is needed that considers contaminants, authentic libraries,
  and tandem MS data — khipu only provides generalized preannotation and leaves those
  steps to downstream tools.
- When the input is raw MS data or a chromatogram rather than a preprocessed feature
  table — khipu requires an asari-style feature table and does not perform peak picking
  itself.
- When the isotope/adduct search patterns do not match the biochemical experiment
  and no custom patterns can be supplied — default patterns may miss or mis-group
  ions.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: khipu
  role: Core Python 3 package implementing the generalized tree (khipu/Weavor) algorithms;
    used as a CLI (khipu -i … -o …) or as a library in Jupyter notebooks to perform
    the isotope/adduct search, network-to-DAG conversion, grid alignment, and neutral
    mass regression.
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Upstream preprocessing tool that generates the feature tables consumed by
    khipu (all test and yeast datasets were processed with asari 1.9.2); asari itself
    uses khipu internally for preannotation.
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Dependency providing the search functions used to find mass-difference-matched
    feature pairs in the isotope/adduct search step.
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: metDataModel
  role: Provides the 'empirical compound' data model that structures the JSON output
    of the khipu trees.
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:2b0c107e1e71b0907caf48ed0dc412eb6a3d28ae9edc6288e89c70c865180ca1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - asari
  - asari 1.9.2
  - khipu
  - mass2chem
  - metDataModel
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# isotope-adduct-tree-annotation

## Summary

Khipu-based preannotation that converts an LC-MS feature table into empirical-compound trees by searching isotope mass differences (1.0034 Da for 13C/12C) and adduct differences (e.g. 22.9893 Da for Na+) among feature m/z values, converting the mass distance network into a directed acyclic graph, and aligning it to a 2-tier adduct/isotope grid to infer neutral M0 masses.

## When to use

Use when you have an asari- or other-preprocessed LC-MS feature table (tab-delimited feature ID, m/z, rtime, intensities) in positive or negative ESI mode and need to group degenerate ions into empirical compounds, including stable isotope tracing (e.g. U-13C-labeled E. coli) or custom adduct experiments.

## When NOT to use

- When complete annotation is needed that considers contaminants, authentic libraries, and tandem MS data — khipu only provides generalized preannotation and leaves those steps to downstream tools.
- When the input is raw MS data or a chromatogram rather than a preprocessed feature table — khipu requires an asari-style feature table and does not perform peak picking itself.
- When the isotope/adduct search patterns do not match the biochemical experiment and no custom patterns can be supplied — default patterns may miss or mis-group ions.

## Inputs

- tab-delimited LC-MS feature table (first columns: feature ID, m/z, rtime, followed by intensity columns), e.g. asari 1.9.2 output such as testdata/ecoli_pos.tsv
- optional user-supplied isotope and adduct search pattern lists for custom experiments (supplied via demo notebooks or library call)
- optional CLI parameters: mode (pos/neg), ppm mass precision, rtol retention-time tolerance, start/end intensity columns, output prefix

## Outputs

- khipu transparent JSON file of empirical compound trees (e.g. this_test.json) using the empCpd data model from metDataModel
- tab-delimited table of annotated empirical compounds (e.g. this_test.tsv) with features, m/z values, and intensities
- inferred neutral M0 masses per empirical compound, and isotope-index assignments (e.g. 13C/12C*1, *2, ...) per adduct trunk

## How to apply

Load the tab-delimited feature table where the first columns are feature ID, m/z, rtime followed by intensities, specifying the start and end intensity columns if needed. Search all pairs of feature m/z values against the default isotope patterns (1.0034 Da per 13C/12C step) and adduct patterns (e.g. 22.9893 Da for Na+, 1.007276 for M+H, 18.033826 for M+NH4), connect matched pairs into a mass distance network, partition it into connected subnetworks, and convert each subnetwork into a directed acyclic graph so erroneous edges that cannot satisfy a rooted tree are weeded out. Optimize the adduct trunk, assign each isotopic branch (groups of isotope edges sharing nodes, each belonging to one adduct type) to the trunk, re-align isotopes to the khipu grid, and infer the neutral M0 mass by linear regression from the matched m/z values. Run as a standalone CLI with a mass precision in ppm (--ppm) and retention-time tolerance (--rtol) if needed, or as a Python 3 library in a Jupyter notebook; results are written as JSON (khipu transparent empCpd format) and tab-delimited tables. For isotope tracing or chemical labeling experiments, overwrite the default isotope/adduct search patterns with experiment-specific ones via the demo notebooks, and for chemical derivatization link separate khipu trees by the m/z shift from derivatization.

## Related tools

- **khipu** (Core Python 3 package implementing the generalized tree (khipu/Weavor) algorithms; used as a CLI (khipu -i … -o …) or as a library in Jupyter notebooks to perform the isotope/adduct search, network-to-DAG conversion, grid alignment, and neutral mass regression.) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Upstream preprocessing tool that generates the feature tables consumed by khipu (all test and yeast datasets were processed with asari 1.9.2); asari itself uses khipu internally for preannotation.) — https://github.com/shuzhao-li/asari
- **mass2chem** (Dependency providing the search functions used to find mass-difference-matched feature pairs in the isotope/adduct search step.) — https://github.com/shuzhao-li-lab/mass2chem
- **metDataModel** (Provides the 'empirical compound' data model that structures the JSON output of the khipu trees.) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pip install khipu-metabolomics; khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- On the E. coli ESI+ test table (testdata/ecoli_pos.tsv, 3,602 features), the run must yield exactly 548 empirical compounds with 1,745 features grouped and 1,857 singletons; 445 of the 548 compounds should have multiple isotopes.
- On the yeast MassIVE MSV000087434 tables, default khipu must produce 1,775 empirical compounds in ESI+ and 908 in ESI–; extending the adduct list should increase explained features from 6,310 to 8,049 (ESI+) and 2,601 to 2,912 (ESI–) without changing the empirical compound counts.
- For the U-13C-labeled E. coli compound with inferred neutral mass 187.1686, the isotope envelope must peak at 13C/12C*9 (M9) in labeled samples and at M0 in unlabeled samples, matching acetylspermidine (C9H21N3O, 9 carbons).
- Schema check: the JSON output parses as khipu empCpd trees and each tree can be visualized as a 2-tier adduct/isotope structure; every empirical compound tree has an inferred neutral mass obtained by linear regression from its matched m/z values.
- Isotope-index extraction must be possible per compound: within each khipu, isotope edges form groups by shared nodes and each group belongs to one adduct type, so the per-compound isotope index (e.g. 13C/12C*1, *2) can be tabulated.

## Limitations

- Benchmarking and systematic improvements are deferred to future work; the method has not yet been benchmarked against alternative tools in the article.
- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved — khipu is a preannotation step, not a complete annotation solution.
- Complete annotation still requires additional resources such as contaminants, authentic libraries, and tandem mass spectrometry data beyond khipu preannotation.
- Many current tools take a network approach but depend on assumed base ions or formulas; khipu's tree approach differs, but its default isotope/adduct patterns must be adapted for non-standard biochemical or derivatization experiments.
- Application to chemical derivatization experiments is planned but not yet released.

## Evidence
