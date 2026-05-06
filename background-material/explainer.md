# background-material Explainer

## Problem

Installed web apps cannot use OS-native translucent/material window frames, and HTML elements cannot request native material backplates.

## Goals

- Allow installed web apps to opt into material-backed window frames.
- Provide a CSS property for material-backed page and element surfaces.
- Use abstract material names, not OS-specific names.
- Preserve fallback behavior on platforms that do not support real materials.

## Non-goals

- Defining spatial layout.
- Exposing sampled environment data to script.
- Styling browser chrome.
- Standardizing vendor material names.

## Proposal

Manifest:

```json
{
  "display": "standalone",
  "background_material": "regular"
}
````

CSS:

```css
:root {
  background-color: transparent;
  background-material: regular;
}

.panel {
  background-material: regular;
  border-radius: 12px;
}
```

Initial values:

```css
none | transparent | thin | regular | thick
```

## Why manifest + CSS?

The manifest opt-in gates top-level window transparency because it affects OS-level app chrome and has spoofing risk.

CSS controls the actual visual treatment after the app has opted in.

## Relationship to backdrop-filter

[`backdrop-filter`](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter) samples DOM content behind an element.

`background-material` samples the OS/environment surface behind the web surface where the UA supports that composition.

For normal opaque browser tabs, authors should generally use [`backdrop-filter`](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter).

## Prior work

* [Microsoft Edge Materials in Web Applications explainer](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/Materials/explainer.md)
* [WebSpatial `background-material` API](https://webspatial.dev/docs/api/react-sdk/css-api/background-material)
* [CSS Spatial Layout Module Level 1 explainer](https://webkit.github.io/explainers/css-spatial/Overview.html)
* [Web App Manifest](https://www.w3.org/TR/appmanifest/)
* [WICG](https://wicg.io/)

## Open questions

* Final value vocabulary.
* Whether per-element rendering is required or optional.
* Fallback behavior when real materials are unavailable.
* Interaction with [`prefers-reduced-transparency`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-transparency).
* Contrast/accessibility guidance.
* Feature detection using [`@supports`](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports) and/or a media feature.

