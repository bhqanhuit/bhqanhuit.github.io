# Academic Portfolio — Bui Huynh Quoc Anh

Personal research portfolio: bio, news, publications, and technical focus areas.

Live at **https://bhqanhuit.github.io**

## About me

Undergraduate student at the University of Information Technology, VNU-HCM (UIT) and visiting
research student at Temasek Laboratories @ SUTD, working with Assoc. Prof. Ngai-Man Cheung.
My research centers on **AI security** and **trustworthy machine learning**, particularly the
security and reliability of deployed AI systems.

- [Google Scholar](https://scholar.google.com/citations?user=V5y9YfIAAAAJ&hl=en)
- [GitHub](https://github.com/bhqanhuit)
- [LinkedIn](https://www.linkedin.com/in/quocanhad123/)

## Contents

| Path | Purpose |
| --- | --- |
| `Academic Portfolio.dc.html` | The portfolio page — template plus its logic |
| `support.js` | Rendering runtime (generated; not edited by hand) |
| `uploads/` | Publication teaser figures |
| `portrait-circle.png` | Profile photo |

## Running locally

The page is static but loads React from a CDN at runtime, so serve it over HTTP rather than
opening the file directly:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000/Academic%20Portfolio.dc.html>.

## How it is built

The page is authored as a Claude Design document. Each file pairs a declarative template
(inside `<x-dc>`) with a `Component` class that supplies the data the template renders against.
Interpolation (`{{ … }}`) resolves property paths only — no expressions — so derived values such
as labels, conditional styles, and event handlers are computed in `renderVals()` and passed to
the template as plain keys. Theming is a `data-theme` attribute switching a set of CSS custom
properties, and the layout is print-aware for exporting a CV.
