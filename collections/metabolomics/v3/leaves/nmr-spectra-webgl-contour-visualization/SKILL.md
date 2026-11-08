---
name: nmr-spectra-webgl-contour-visualization
description: Use when an already-processed 2D or pseudo-3D NMR spectrum needs interactive
  contour or 3D surface-mesh rendering that is architecture- and OS-independent, with
  no installation required.
when_to_use_negative:
- Input is a raw Bruker time-domain FID (fid, ser) that still requires zero-filling,
  apodization, Fourier transformation, and phase correction before it can be rendered
  as a contour or mesh.
- Task requires quantitative deconvolution or peak parameters rather than visual rendering
  — use DEEP Picker / Voigt Fitter integration instead.
- Rendering must be done by a non-browser desktop tool or must run where WebGL in
  a recent web browser is unavailable.
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Browser-based tool that performs the WebGL 2D contour rendering and 3D surface
    mesh visualization of processed 2D/pseudo-3D NMR spectra
  repo: https://github.com/lidawei1975/colmarvista
- name: JavaScript
  role: Implementation basis that makes the advanced graphics (contours and mesh rendering)
    computer-architecture and operating-system independent
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:1f9c32fb596ad16e2d7fde4c91a49f35476ecebdfe0326ed49ffac2f74fe1e04
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - JavaScript
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# nmr-spectra-webgl-contour-visualization

## Summary

Render already-processed 2D and pseudo-3D NMR spectra as WebGL 2D contour plots and 3D surface meshes inside COLMARvista, a JavaScript-based browser tool, and export the two renderings as images for visual comparison of peak content.

## When to use

Use when an already-processed 2D or pseudo-3D NMR spectrum needs interactive contour or 3D surface-mesh rendering that is architecture- and OS-independent, with no installation required.

## When NOT to use

- Input is a raw Bruker time-domain FID (fid, ser) that still requires zero-filling, apodization, Fourier transformation, and phase correction before it can be rendered as a contour or mesh.
- Task requires quantitative deconvolution or peak parameters rather than visual rendering — use DEEP Picker / Voigt Fitter integration instead.
- Rendering must be done by a non-browser desktop tool or must run where WebGL in a recent web browser is unavailable.

## Inputs

- Processed 2D NMR spectrum file (already Fourier-transformed and phase-corrected)
- Processed pseudo-3D NMR series data
- Contour threshold and contour level settings matched to the spectrum's intensity range

## Outputs

- 2D WebGL contour plot image (PNG or SVG screenshot)
- 3D surface mesh rendering image (PNG or equivalent screenshot)
- Visual confirmation that contour and mesh renderings show equivalent peak content

## How to apply

Load the processed 2D spectrum file into COLMARvista (via the web server https://lidawei1975.github.io/colmarvista/ or a downloaded local copy with index.html opened in the browser). Set the contour threshold and contour levels appropriate for the spectrum's intensity range, then render the WebGL 2D contour plot and export it as a PNG or SVG screenshot. Switch the renderer to 3D surface mesh mode and render the same spectrum as a mesh, saving the mesh image (e.g. PNG screenshot). Because COLMARvista is based on JavaScript, both renderings run in any recent web browser with no operating-system-dependent installation; verify the contour and mesh renderings represent equivalent peak content (same peak positions and intensities). If running from a local download, note that WebWorker/WebAssembly features require browser configuration such as Chrome's --allow-file-access-from-files flag.

## Related tools

- **COLMARvista** (Browser-based tool that performs the WebGL 2D contour rendering and 3D surface mesh visualization of processed 2D/pseudo-3D NMR spectra) — https://github.com/lidawei1975/colmarvista
- **JavaScript** (Implementation basis that makes the advanced graphics (contours and mesh rendering) computer-architecture and operating-system independent)

## Examples

```
npm start  # launches http-server at http://127.0.0.1:8080 for COLMARvista; then open index.html in a recent web browser to render contours and 3D mesh
```

## Evaluation signals

- The 2D contour plot renders in the browser with a threshold/level setting that makes the spectrum's peak pattern visible without clipping or noise flooding.
- The same spectrum renders as a 3D surface mesh after switching renderer modes.
- Exported contour and mesh images can be saved as PNG/SVG screenshots.
- The two renderings show equivalent peak content — the same peak positions and relative intensities are represented in both.
- No operating-system-dependent installation step was needed; rendering completed in a recent web browser.

## Limitations

- Local (offline) use via index.html requires manual browser configuration for WebWorker/WebAssembly (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari 'Disable Local File Restrictions'), which the README flags as a security risk.
- Rendering is limited to what WebGL in a recent web browser supports; very large spectra may strain browser performance.
- No changelog was found for the repository, so renderer behavior may change between versions without a documented record.

## Evidence
