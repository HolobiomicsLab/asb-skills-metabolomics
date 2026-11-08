---
name: java-scientific-computing
description: Use when implementing statistical analyses (e.g. B-spline mutual information,
  hierarchical clustering, Pearson/Spearman correlations) in Java on numeric expression
  matrices, or when reusing MOG's Java analysis modules.
when_to_use_negative:
- The analysis requires interactive R-based statistical packages rather than a standalone
  Java implementation — MOG only optionally sends data subsets to R
- The goal is exploratory visualization through MOG's GUI rather than programmatic
  implementation of a new Java analysis module
- Input is not tabular numeric expression data with associated feature/sample identifiers
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: MetaOmGraph
  role: Java workbench whose analysis modules (including the Mutual Information module)
    this skill implements or extends
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Implementation language for MOG and its statistical analysis modules
- name: hierarchical-clustering-java
  role: Java library MOG uses for agglomerative hierarchical clustering with single/complete/average
    linkage on a distance matrix
  repo: https://github.com/lbehnke/hierarchical-clustering-java
- name: Apache Commons Math
  role: Java math library dependency used by MOG for statistical computations
  repo: https://github.com/apache/commons-math
- name: JFreeChart
  role: Java charting library dependency used by MOG for visualization of analysis
    results
  repo: https://github.com/jfree/jfreechart
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:20b9c27c2f8d1a4253e6edc2643eb879b1a5ac45329f7c32b268e202387895ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Apache Commons Math
  - JFreeChart
  - Java
  - MOG
  - MetaOmGraph
  - hierarchical-clustering-java
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# java-scientific-computing

## Summary

Implement scientific statistical analyses in Java, following the MetaOmGraph (MOG) pattern: a standalone Java workbench that performs co-expression, mutual information, differential expression and clustering analyses on large expression matrices using libraries like Apache Commons Math and hierarchical-clustering-java.

## When to use

Use when implementing statistical analyses (e.g. B-spline mutual information, hierarchical clustering, Pearson/Spearman correlations) in Java on numeric expression matrices, or when reusing MOG's Java analysis modules.

## When NOT to use

- The analysis requires interactive R-based statistical packages rather than a standalone Java implementation — MOG only optionally sends data subsets to R
- The goal is exploratory visualization through MOG's GUI rather than programmatic implementation of a new Java analysis module
- Input is not tabular numeric expression data with associated feature/sample identifiers

## Inputs

- Numeric expression matrix (feature rows x sample columns, e.g. CSV/tab-delimited gene expression data)
- Distance matrix and cluster-name array for hierarchical clustering
- Java Runtime Environment 8 (or higher); R 3.4+ optional

## Outputs

- Pairwise mutual information values (e.g. mi_matrix.csv of feature-pair MI estimates)
- Cluster hierarchy (Cluster object renderable via DendrogramPanel)
- Statistical analysis results embedded in a savable/shareable MOG project file

## How to apply

Clone the MetaOmGraph source with `git clone https://github.com/urmi-21/MetaOmGraph.git` and build with Maven (the repo runs Maven CI). Load a numeric expression matrix (feature rows x sample columns) and implement analyses as Java modules: for pairwise mutual information, follow Daub et al.'s estimator, which estimates each feature's marginal density with B-splines and exploits the property that I(X;Y)=0 exactly when X and Y are statistically independent. For clustering, pass a distance matrix and cluster-name array to hierarchical-clustering-java's DefaultClusteringAlgorithm with a linkage strategy (single, complete, or average). Reuse established Java libraries rather than reimplementing primitives — MOG itself depends on JFreeChart, Apache Commons Math, the Nitrite database, and Colorbrewer. Require only Java Runtime Environment 8+ so the module runs standalone; optionally bridge to R 3.4+ when additional analyses are needed.

## Related tools

- **MetaOmGraph** (Java workbench whose analysis modules (including the Mutual Information module) this skill implements or extends) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Implementation language for MOG and its statistical analysis modules)
- **hierarchical-clustering-java** (Java library MOG uses for agglomerative hierarchical clustering with single/complete/average linkage on a distance matrix) — https://github.com/lbehnke/hierarchical-clustering-java
- **Apache Commons Math** (Java math library dependency used by MOG for statistical computations) — https://github.com/apache/commons-math
- **JFreeChart** (Java charting library dependency used by MOG for visualization of analysis results) — https://github.com/jfree/jfreechart

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git && mvn package
```

## Evaluation signals

- All computed MI values are non-negative, and pairs of simulated-random/independent features yield MI values near zero
- B-spline MI estimates agree with an independent reference MI implementation across all feature pairs within a stated tolerance
- Clustering output reproduces the expected dendrogram for a known toy distance matrix (e.g. the 6-object example in hierarchical-clustering-java)
- Maven CI build passes and the module runs on Java Runtime Environment 8+
- Analysis results are written to the expected CSV format (e.g. mi_matrix.csv) with a validation report of the comparison summary

## Limitations

- The B-spline MI estimator's accuracy depends on B-spline density estimation choices; agreement with reference implementations should be checked per dataset
- Standalone Java implementation lacks the breadth of R statistical packages; MOG only optionally bridges to R 3.4+ for additional analyses
- MOG requires Java Runtime Environment 8 or higher and the hierarchical clustering library ships only single, complete, and average linkage strategies

## Evidence

- [intro] MOG is a Java software for scientific analysis: "we present MetaOmGraph (MOG), a Java software, to interactively explore and visualize large expression datasets"
- [readme] MOG MI module citation of Daub et al.: [evidence span withheld — anchor status: external_doc]
- [readme] Java libraries MOG depends on: "* [JFreeChart](https://github.com/jfree/jfreechart)
* [Apache Commons Math](https://github.com/apache/commons-math)"
- [readme] Hierarchical clustering usage pattern: [evidence span withheld — anchor status: external_doc]
- [readme] Prerequisites for running MOG: "* Java Runtime Environment 8 (or higher)
* R 3.4 (or higher) [optional]"
- [methods] Cloning the source for reuse: "git clone https://github.com/urmi-21/MetaOmGraph.git"
