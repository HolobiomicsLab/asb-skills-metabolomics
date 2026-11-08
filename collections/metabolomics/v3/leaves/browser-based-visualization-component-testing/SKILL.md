---
name: browser-based-visualization-component-testing
description: Use when the goal is to embed or reuse an interactive 3D ordination visualization
  (e.g. PCoA plots) inside another tool or web page, and you need to confirm it renders
  correctly in a browser.
when_to_use_negative:
- Input is raw sequencing reads or a feature table — this skill assumes ordination
  results already computed upstream (PCoA on a beta diversity distance matrix), not
  raw data processing.
- The target environment must render WebGL/HTML5 in a specific browser; per the paper's
  requirements, the graphical user interface requires Chrome, so this skill is not
  the right choice when the consumer must use a browser without WebGL support or a
  non-Chrome engine.
- The task is a static publication figure rather than an interactive embedded component;
  EMPeror's value here is interactive manipulation and metadata-driven recoloring,
  which a static image does not need.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: EMPeror
  role: Generates the interactive 3D PCoA visualization from QIIME principal coordinates
    and metadata mapping files and produces the embeddable web-page deliverable.
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Supplies the upstream ordination (principal coordinates analysis) whose output
    file is one of the two inputs to EMPeror; also listed as a requirement (python
    libraries only).
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:e454872493910995758a22d95f56614458afecc8465bd543cda007ede2b59b68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# browser-based-visualization-component-testing

## Summary

Validates that EMPeror's interactive 3D principal-coordinates visualization can be generated from QIIME PCoA and mapping files and embedded as a reusable component in a host web page, without extra plugins.

## When to use

Use when the goal is to embed or reuse an interactive 3D ordination visualization (e.g. PCoA plots) inside another tool or web page, and you need to confirm it renders correctly in a browser.

## When NOT to use

- Input is raw sequencing reads or a feature table — this skill assumes ordination results already computed upstream (PCoA on a beta diversity distance matrix), not raw data processing.
- The target environment must render WebGL/HTML5 in a specific browser; per the paper's requirements, the graphical user interface requires Chrome, so this skill is not the right choice when the consumer must use a browser without WebGL support or a non-Chrome engine.
- The task is a static publication figure rather than an interactive embedded component; EMPeror's value here is interactive manipulation and metadata-driven recoloring, which a static image does not need.

## Inputs

- QIIME principal coordinates file (PCoA output)
- Metadata mapping file
- Host web page in which the EMPeror visualization will be embedded

## Outputs

- Interactive, hardware-accelerated 3D principal-coordinates visualization rendered as a web page component (HTML5/WebGL)
- Embedded page loadable in Chrome without extra plugins

## How to apply

Download the deposited example inputs — the QIIME principal coordinates file and metadata mapping file — from the GigaScience Database or the EMPeror FTP site referenced in the Availability of Supporting Data section. Run the EMPeror command line interface on these two files to generate the interactive, hardware-accelerated 3D web visualization (implemented with HTML5, WebGL, JavaScript and Python, requiring Chrome, QIIME python libraries, NumPy, BIOM 1.1.0 and PyCogent). Embed the generated output into a host HTML page as a reusable component, following the paper's feature that EMPeror visualizations can be 'easily embedded into other tools, such as Evident'. Open the embedded page in Chrome (the browser listed as a requirement) and verify the WebGL 3D PCoA plot renders sample spheres. Confirm output size stays small enough for sharing via email or web page — the paper reports 26 MB for 5,740 samples and 120 metadata columns versus 1.85 GB in KiNG — so embedded visualizations load in seconds rather than minutes.

## Related tools

- **EMPeror** (Generates the interactive 3D PCoA visualization from QIIME principal coordinates and metadata mapping files and produces the embeddable web-page deliverable.) — http://emperor.colorado.edu
- **QIIME** (Supplies the upstream ordination (principal coordinates analysis) whose output file is one of the two inputs to EMPeror; also listed as a requirement (python libraries only).)

## Examples

```
python scripts/embed_and_validate.py --pc qiime_pcoa.txt --map metadata_map.txt --host embedded_emperor.html
```

## Evaluation signals

- The embedded page loads in Chrome and displays the 3D WebGL PCoA plot with sample spheres rendered (no plugin prompts or missing-file errors).
- The generated visualization file size is on the order of tens of megabytes for thousands of samples and many metadata columns — e.g. ~26 MB for 5,740 samples and 120 metadata columns — not gigabytes as in KiNG.
- Page load time is seconds, not many minutes, when opened in the browser (EMPeror's stated advantage over KiNG).
- Interactive controls respond: principal coordinate axes can be switched, metadata-driven recoloring and point visibility toggles apply, and gradient/categorical/biplot/procrustes/jackknifed views can be triggered from the UI.
- The component functions when served from or embedded into a different host page, confirming the 'easily embedded into other tools' claim rather than only working as a standalone HTML file.

## Limitations

- The command line interface runs only on OS X (10.6 and higher) and Linux; no Windows support is stated.
- Requires Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent — dependencies on an end-of-life Python version and a single named browser.
- No explicit limitations section is given in the article text, and no archival DOI or accession IDs are provided for the supporting example files (they are only referenced to the GigaScience Database and EMPeror FTP site).
- Only a project home page URL (http://emperor.colorado.edu) is given; no code repository URL is cited in this text.

## Evidence

- [abstract] Embedding claim: "ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [abstract] Output size and scaling: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] Implementation stack and dependencies: "EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python, and tightly"
- [discussion] Software requirements listing: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [discussion] Data availability: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Browser requirement (CLI scope): "OS X (10.6 and higher) and Linux only for the command line interface."
