---
name: microbiome-metadata-mapping-integration
description: Use when you have QIIME PCoA coordinates and a metadata mapping file
  and need an interactive, embeddable 3D ordination plot (Chrome/WebGL) shared via
  email or a web page.
when_to_use_negative:
- Input is raw sequencing data or a feature table rather than ordination results —
  PCoA of a beta diversity distance matrix must be run first.
- You need a static publication-quality 2D figure rather than an interactive web-browser
  3D plot shared via email or web page.
- The execution environment is Windows-only — the EMPeror command line interface is
  stated to run on OS X (10.6 and higher) and Linux only.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Generates the interactive, hardware-accelerated WebGL 3D PCoA visualization
    from the principal coordinates file and metadata mapping file and produces the
    embeddable web-page component
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Supplies the upstream principal coordinates files and metadata mapping files
    that EMPeror consumes (python libraries only required)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:aa710071a955d1371a8ec02568c91f30b0926cef680fe3068f01631779227a58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# microbiome-metadata-mapping-integration

## Summary

A skill for turning QIIME principal coordinates files plus metadata mapping files into an interactive, hardware-accelerated WebGL 3D PCoA visualization (EMPeror) that can be embedded as a reusable component in a host web page or tool such as Evident, without extra browser plugins.

## When to use

Use when you have QIIME PCoA coordinates and a metadata mapping file and need an interactive, embeddable 3D ordination plot (Chrome/WebGL) shared via email or a web page.

## When NOT to use

- Input is raw sequencing data or a feature table rather than ordination results — PCoA of a beta diversity distance matrix must be run first.
- You need a static publication-quality 2D figure rather than an interactive web-browser 3D plot shared via email or web page.
- The execution environment is Windows-only — the EMPeror command line interface is stated to run on OS X (10.6 and higher) and Linux only.

## Inputs

- QIIME principal coordinates file (from PCoA of a beta diversity distance matrix)
- QIIME metadata mapping file (sample metadata columns)
- Optional jackknifed-resampled PCoA results or second coordinate set for procrustes/biplots

## Outputs

- Embedded, self-contained interactive WebGL 3D PCoA visualization web page (MB-scale, e.g. embedded_emperor.html)
- Reusable visualization component embeddable within a host tool such as Evident

## How to apply

Run the EMPeror command line interface on the QIIME principal coordinates file together with the metadata mapping file to generate a self-contained interactive 3D visualization deliverable in a web page. Then embed the generated output into a host web page (Evident-style) as a reusable visualization component; EMPeror's small output size (26 MB for 5,740 samples and 120 metadata columns, i.e. 1.3% of the equivalent 1.85 GB KiNG files) makes the embedded page shareable by email or web. Use the EMPeror controllers to recolor or hide points dynamically as a function of metadata, visualize gradients and categorical data, switch principal coordinates axes, and optionally add biplots, procrustes analysis plots, or jackknifed beta diversity plots for statistical confidence in clustering. The host page must be opened in Chrome per the stated requirements (Python 2.7, Chrome, QIIME python libraries only, NumPy, BIOM 1.1.0, PyCogent).

## Related tools

- **EMPeror** (Generates the interactive, hardware-accelerated WebGL 3D PCoA visualization from the principal coordinates file and metadata mapping file and produces the embeddable web-page component) — http://emperor.colorado.edu
- **QIIME** (Supplies the upstream principal coordinates files and metadata mapping files that EMPeror consumes (python libraries only required))

## Examples

```
python make_emperor.py -i pcoa.txt -m mapping.txt
```

## Evaluation signals

- embedded_emperor.html loads in Chrome and the WebGL 3D PCoA plot renders sample spheres
- Embedded page requires no extra browser plugins and opens directly as a web page or email attachment
- Output file remains MB-scale; expect ~26 MB for 5,740 samples and 120 metadata columns (1.3% of the KiNG-equivalent size)
- Sample points recolor and toggle visibility dynamically as a function of metadata mapping-file columns
- Page loads in seconds rather than the many minutes reported for KiNG at equivalent scale

## Limitations

- Command line interface limited to OS X (10.6 and higher) and Linux only; no Windows support stated
- Depends on Python 2.7 (end-of-life) plus Chrome for the graphical user interface, suggesting untested compatibility with other browsers
- No version number or code repository URL stated for the software; only a project home page URL (http://emperor.colorado.edu)
- Supporting example files carry no archival DOI or accession IDs in the text

## Evidence

- [abstract] abstract: "ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [discussion] discussion: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
