# install-free-pwa Explainer

## Problem

Web apps must be installed to access standalone window features (custom title bar, file handlers, share targets, spatial windows). Install is a high-friction, up-front commitment made before the user has experienced any value, which sacrifices the web's core advantage: zero-install, link-launchable reach.

## Goals

- Allow web apps to render in a standalone-style window without prior install.
- Keep the URL as the primary launch path.
- Treat install as a retention upgrade, not an entry barrier.
- Make "try before you buy" first-class for app-grade web experiences, including spatial.

## Non-goals

- Removing or replacing the install action.
- Granting elevated permissions (push, badging, persistent storage) without an explicit install.
- Defining new window chrome or layout primitives.
- Changing the security model of the Web App Manifest.

## Proposal

Manifest opt-in that lets the UA render an installable PWA in its standalone display mode on first navigation, without requiring the user to install first.

Manifest:

```json
{
  "display": "standalone",
  "install_free": true
}
```

Behavior:

- The page declares an installable manifest as it does today.
- When `install_free` is `true`, a supporting UA may launch the page in standalone chrome from a normal link click, in addition to (not instead of) the existing post-install launch path.
- Install remains available and unlocks persistence, app drawer entry, OS-level launch, and elevated capabilities.

## Why a manifest opt-in?

Standalone chrome (no URL bar) carries spoofing risk. The opt-in keeps the same security envelope as an installed PWA — the manifest, HTTPS, and scope rules still gate the experience — but separates the windowing trigger from the install ceremony.

## Relationship to install

- Install today: required for standalone window, required for capabilities, required for persistence.
- Install with `install_free`: optional for standalone window, still required for capabilities and persistence.
- Install becomes a signal of intent from an already-engaged user, not a precondition for the experience.

## Prior work

- [Web App Manifest](https://www.w3.org/TR/appmanifest/)
- [Google Play Instant Apps](https://developer.android.com/topic/google-play-instant) (deprecated; same UX goal on Android)
- [Apple App Clips](https://developer.apple.com/app-clips/)
- [Window Controls Overlay](https://wicg.github.io/window-controls-overlay/)
- Alex Russell, ["Progressive Apps: Escaping Tabs Without Losing Our Soul"](https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/) (2015)
- [WICG](https://wicg.io/)

## Open questions

- Storage and quota: same as installed PWA, or ephemeral until install?
- Capability gating: which APIs (push, badging, file handlers, protocol handlers) remain install-only?
- Session lifetime when the user navigates away or closes the window.
- UA affordances to distinguish an install-free standalone session from an installed app, and to surface the install promotion path.
- Interaction with `display_override` and existing display modes.
- Spatial-specific: volume window lifecycle, scope handling across spatial transitions, and how the install promotion is presented inside an XR shell.
- Feature detection from script and from CSS.
