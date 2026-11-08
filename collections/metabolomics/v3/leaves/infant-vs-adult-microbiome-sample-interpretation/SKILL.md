---
name: infant-vs-adult-microbiome-sample-interpretation
description: Use when exploratory ordination of a large combined microbiome PCoA (thousands
  of samples, dozens of metadata columns) must reveal infant-vs-adult age differences
  via gradient or discrete metadata coloring.
when_to_use_negative:
- Input is raw sequence data or a feature/Biom table that has not yet been collapsed
  to a PCoA ordination of beta diversity distances — EMPeror consumes principal coordinates
  and mapping files, not FASTQ or Biom
- The analysis is a downstream statistical test or supervised classification of age;
  this skill is exploratory visualization and coloring, not hypothesis testing
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: EMPeror
  role: Interactive WebGL 3D PCoA visualization with gradient and discrete metadata
    coloring controllers used to expose infant-versus-adult age patterns
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Supplies the principal coordinates file and metadata mapping file consumed
    by EMPeror, and its Python libraries are a stated requirement for the EMPeror
    command line interface
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:eea798b92fc43a965472dacd74bd9224bb51c9e2640ccc6f1872dfeedb2ce4ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# infant-vs-adult-microbiome-sample-interpretation

## Summary

Interactive 3D PCoA visualization and metadata-integrated coloring in EMPeror to reveal infant-versus-adult age patterns across combined microbiome datasets.

## When to use

Use when exploratory ordination of a large combined microbiome PCoA (thousands of samples, dozens of metadata columns) must reveal infant-vs-adult age differences via gradient or discrete metadata coloring.

## When NOT to use

- Input is raw sequence data or a feature/Biom table that has not yet been collapsed to a PCoA ordination of beta diversity distances — EMPeror consumes principal coordinates and mapping files, not FASTQ or Biom
- The analysis is a downstream statistical test or supervised classification of age; this skill is exploratory visualization and coloring, not hypothesis testing

## Inputs

- QIIME principal coordinates file (PCoA axes) for the combined microbiome dataset
- QIIME metadata mapping file with age and other sample metadata (120 columns for the 5,740-sample combined dataset)

## Outputs

- Interactive WebGL 3D PCoA visualization opened in Chrome (EMPeror HTML output)
- Exported figure image showing metadata-integrated coloring (e.g. figure_1b_age_coloring.png) that reveals infant versus healthy adult samples

## How to apply

Download the example QIIME principal coordinates file and metadata mapping file for the combined 5,740-sample / 120-column dataset (GigaScience Database or the EMPeror FTP site), then run the EMPeror command line interface (OS X 10.6+ / Linux only, with Python 2.7, Chrome, QIIME python libraries, NumPy, BIOM 1.1.0 and PyCogent) to generate an interactive WebGL 3D PCoA visualization opened in Chrome. Use EMPeror's gradient-coloring controller so each sample's color is a function of its position in the age metadata gradient, and apply the discrete/categorical coloring variant, to visualize the infant time-series samples alongside the samples from healthy human adults. The workflow builds on PCoA of beta diversity distance matrices as the underlying dimensionality reduction, and the lightweight output file format is what makes this practical for 5,740 samples — EMPeror stores the result at 26 MB versus 1.85 GB in KiNG, reducing loading times and memory footprint so the interaction stays responsive. Export a rendering of the panel showing infant-vs-adult separation (e.g. as figure_1b_age_coloring.png), validating visually against the published Figure 1B panels.

## Related tools

- **EMPeror** (Interactive WebGL 3D PCoA visualization with gradient and discrete metadata coloring controllers used to expose infant-versus-adult age patterns) — http://emperor.colorado.edu
- **QIIME** (Supplies the principal coordinates file and metadata mapping file consumed by EMPeror, and its Python libraries are a stated requirement for the EMPeror command line interface)

## Examples

```
python /path/to/EMPeror/make_3d_plots.py -i pcoa.txt -m mapping.txt -o emperor_out; open emperor_out/index.html in Chrome
```

## Evaluation signals

- The exported plot must reproduce the reported result: integrating the age metadata gradient into the coloring clearly shows the age differences between infant samples and samples belonging to healthy human adults (expert visual comparison against the published Figure 1B panels)
- The output must be a loadable PNG file (e.g. figure_1b_age_coloring.png)
- Both coloring variants must be demonstrable: the gradient-coloring controller applied to the age metadata and the discrete/categorical metadata coloring variant (Figure 1B-1 and Figure 1B-2)
- Input scale check: the combined dataset used is 5,740 samples with 120 metadata columns; the EMPeror output for both discrete and gradient coloring should be ~26 MB, i.e. 1.3% of the 1.85 GB KiNG equivalent, confirming the lightweight output format
- Visualization opens in Chrome with responsive interaction (seconds not minutes of loading) and supports changing the principal coordinates axes and metadata coloring dynamically

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated
- Dependence on Python 2.7, an end-of-life Python version, listed as a requirement
- Graphical user interface requires Chrome, suggesting untested compatibility with other browsers
- No version number stated for Emperor itself, no code repository URL (only a project home page), and no archival DOI or accession IDs for the supporting example files

## Evidence

- [other] When the combined datasets of references [3,20-22] (5,740 samples, 120 metadata columns) are visualized with metadata-integrated coloring in EMPeror, intrinsic age patterns are easily viewed in both panels: [evidence span withheld — anchor status: not_found]
- [other] EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python: "EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python"
- [abstract] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB"
- [abstract] ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates"
- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database, as well as from the EMPeror FTP site: [evidence span withheld — anchor status: not_found]
- [other] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
