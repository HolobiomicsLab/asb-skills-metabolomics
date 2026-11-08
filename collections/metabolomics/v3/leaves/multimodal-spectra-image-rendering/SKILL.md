---
name: multimodal-spectra-image-rendering
description: Use when an already-processed 2D or pseudo-3D NMR spectrum needs interactive
  contour visualization, a pseudo-3D surface-mesh rendering, or exported publication-style
  images of both without any local software installation.
when_to_use_negative:
- The spectrum is raw time-domain data (Bruker fid/ser) that still needs zero-filling,
  apodization, Fourier transformation, or phase correction — use the client-side WebAssembly
  processing workflow first.
- The target is quantitative peak deconvolution/line-shape fitting rather than visual
  inspection — use DEEP Picker / Voigt Fitter or Simple Picker instead.
- A non-browser, natively installed visualization program is required (e.g. in an
  environment without WebGL or a recent browser).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Web-based tool used to render the 2D WebGL contour plot and switch to 3D surface
    mesh mode for the same processed spectrum, and to export/save the images.
  repo: https://github.com/lidawei1975/colmarvista
- name: JavaScript
  role: Underlying language making the advanced graphics (WebGL contours and 3D surface
    mesh rendering) architecture- and OS-independent so they run in any recent browser
    without installation.
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:ac6bbd50b436358c5d3bba2736abf1c0dd9acefa85e2487ea46c4060ec52713e
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - JavaScript
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multimodal-spectra-image-rendering

## Summary

Render processed 2D and pseudo-3D NMR spectra as WebGL contour plots and 3D surface meshes directly in a web browser using COLMARvista, whose JavaScript/WebGL graphics are architecture- and OS-independent, and export the resulting contour and mesh images.

## When to use

Use when an already-processed 2D or pseudo-3D NMR spectrum needs interactive contour visualization, a pseudo-3D surface-mesh rendering, or exported publication-style images of both without any local software installation.

## When NOT to use

- The spectrum is raw time-domain data (Bruker fid/ser) that still needs zero-filling, apodization, Fourier transformation, or phase correction — use the client-side WebAssembly processing workflow first.
- The target is quantitative peak deconvolution/line-shape fitting rather than visual inspection — use DEEP Picker / Voigt Fitter or Simple Picker instead.
- A non-browser, natively installed visualization program is required (e.g. in an environment without WebGL or a recent browser).

## Inputs

- Processed 2D NMR spectrum file (loaded into COLMARvista; no upload needed since data stays client-side)
- Contour threshold and contour level settings appropriate for the spectrum's intensity range

## Outputs

- Exported 2D WebGL contour image (e.g. PNG or SVG screenshot)
- Exported 3D surface mesh image (PNG or equivalent screenshot)

## How to apply

Load the processed 2D spectrum into COLMARvista in a recent web browser (e.g. via https://lidawei1975.github.io/colmarvista/). Set the contour threshold and contour levels appropriate for the spectrum's intensity range, then render the WebGL 2D contour plot and export/save the contour image (e.g. PNG screenshot). Switch the renderer to 3D surface mesh mode and render the same spectrum as a mesh, saving the mesh image (PNG or equivalent screenshot). Verify that the two renderings represent equivalent peak content — the same peak positions and intensities must appear in both the contour and the mesh — because COLMARvista relies on high-performance WebGL contours and 3D surface mesh rendering with synchronous zoom/pan so both views probe the same spectral data.

## Related tools

- **COLMARvista** (Web-based tool used to render the 2D WebGL contour plot and switch to 3D surface mesh mode for the same processed spectrum, and to export/save the images.) — https://github.com/lidawei1975/colmarvista
- **JavaScript** (Underlying language making the advanced graphics (WebGL contours and 3D surface mesh rendering) architecture- and OS-independent so they run in any recent browser without installation.) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- The 2D WebGL contour plot and the 3D surface mesh rendering show equivalent peak content (same peak positions and intensity representation in both).
- Contour threshold and levels are set so peaks across the spectrum's intensity range are visible, not clipped or buried in noise.
- Both rendered images were successfully exported/saved (e.g. PNG screenshots) from the browser.
- Rendering succeeds with no OS-dependent installation, confirming COLMARvista runs in the browser with data staying local (no uploads to the server).

## Limitations

- Graphics require WebGL support in a recent browser; local offline use of index.html may require browser configuration (WebWorker/WebAssembly flags such as --allow-file-access-from-files in Chrome, security.fileuri.strict_origin_policy=false in Firefox, or 'Disable Local File Restrictions' in Safari), which poses a security risk.
- Rendering and export are screenshot-based (e.g. PNG or equivalent); no vector/vector-mesh export format beyond what the tool exposes is described.
- The skill applies to already-processed spectra; raw FID processing and NUS reconstruction are separate COLMARvista functions.

## Evidence
