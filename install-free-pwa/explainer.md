# Install-Free Standalone Web Apps Explainer

## Status

Early draft. This is meant to describe a product/platform need and get feedback on whether the Web App Manifest should support this kind of launch mode.

## Summary

Today, if a web app wants to run outside a browser tab, it usually has to be installed first. That makes sense for some things, but it also creates a lot of friction.

For many apps, especially spatial apps, the standalone window is not really an “extra” after install. It is the main way the app is supposed to be experienced. Asking the user to install before they can even see that experience feels backwards.

The idea here is simple:

> Let an eligible web app open in a standalone window before formal install.

Install would still matter. It would still be the step that gives the app durable identity: launcher entry, app icon, OS registration, background tasks, handlers, and other deeper integrations. But standalone windowing itself should not necessarily require install.

This is especially relevant for WebSpatial, where a web app may need to open as a spatial window or volume to make sense at all.

## Problem

The web has a very good first-run model: click a link and try the thing.

PWAs improve the web by adding app-like behavior, but the install step also changes the first-run model. In practice, install often becomes a gate before the user has experienced the value of the app.

That is a problem for app-like and spatial web experiences. A user may need to see the app in a standalone window before deciding whether it is worth keeping.

Examples:

* a spatial web app that should open as its own panel or volume;
* an app that needs a focused window instead of a browser tab;
* a multi-window web app;
* an app that wants custom window/title bar behavior;
* a web app where the browser tab version is not representative of the real experience.

For these cases, the desired flow is:

1. user opens a link;
2. the app opens in the right presentation mode;
3. if the user likes it, they can install it later.

Install should be a retention step, not a requirement before the first useful experience.

## Goals

* Allow a web app to run in a standalone-style window before formal install.
* Keep URL launch as a first-class path.
* Keep install as an explicit user choice.
* Keep OS integration and elevated capabilities gated on install.
* Support “try before install” for app-like web apps.
* Support spatial web apps that need a standalone spatial surface.
* Reuse existing manifest concepts like scope, display mode, HTTPS, and installability checks.

## Non-goals

* Replacing install.
* Giving apps install-only capabilities without install.
* Changing the web permission model for camera, mic, geolocation, WebXR, WebSpatial, etc.
* Defining new window layout APIs.
* Standardizing any current implementation workaround, like generating a temporary native package.

## Proposal

Add a manifest opt-in that lets a browser/UA open an eligible web app in standalone mode before the app has been formally installed.

Example:

```json
{
  "name": "Example App",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "install_free": true
}
```

`install_free` is just a placeholder name. The exact name is open.

When this is set, the UA may offer a way to open the app in standalone mode before install. That could be a browser UI button, a first-run prompt, a remembered user choice, or some other platform-specific launch surface.

This does not replace normal install. Install is still available, and still unlocks the normal installed-app behavior.

## States

There are three states worth separating:

| State                   | Meaning                                                                                |
| ----------------------- | -------------------------------------------------------------------------------------- |
| Browser tab             | The page runs in normal browser UI.                                                    |
| Install-free standalone | The app runs in a standalone window, but is not formally installed.                    |
| Installed app           | The user has installed the app, so it can get durable app identity and OS integration. |

The important distinction is that install-free standalone is not the same as installed. It gives the app a better presentation surface, not a durable place in the OS.

## Capability boundary

The main question is not whether this mode should get “PWA permissions.” PWAs do not really have one permission bundle. There are normal web permissions, and then there are install/app/OS integration features.

This proposal is only about moving standalone windowing before install.

Things that should be available in install-free standalone mode:

* standalone window presentation;
* no normal browser tab chrome, if the UA provides another origin/runtime affordance;
* spatial window or volume presentation in spatial UAs;
* normal web permissions, using the same permission model as the browser;
* normal origin storage, subject to browser policy;
* normal service worker behavior, subject to browser policy;
* a clear path to install.

Things that should stay install-gated:

* app launcher / Home screen entry;
* app icon;
* OS app registration;
* system settings entry;
* URL handlers;
* protocol handlers;
* file handlers;
* share target;
* app shortcuts;
* badging tied to app identity;
* background tasks that depend on installed-app identity;
* install lifecycle events like `appinstalled`;
* any permission upgrades tied to formal install.

A rough table:

| Capability                   |                Browser tab |                Install-free standalone |                Installed app |
| ---------------------------- | -------------------------: | -------------------------------------: | ---------------------------: |
| URL navigation               |                        Yes |                                    Yes |                          Yes |
| Manifest scope               |                        Yes |                                    Yes |                          Yes |
| Standalone window            |                         No |                                    Yes |                          Yes |
| Spatial window / volume      |                UA-specific |                    Yes, in spatial UAs |          Yes, in spatial UAs |
| Browser chrome removed       |                         No |                Yes, with UA affordance |      Yes, with UA affordance |
| App launcher / Home icon     |                         No |                                     No |                          Yes |
| System settings entry        |                         No |                                     No |                          Yes |
| Durable OS app identity      |                         No |                                     No |                          Yes |
| Install lifecycle            |                         No |                                     No |                          Yes |
| Camera / mic / geolocation   |             Web permission |                         Web permission |               Web permission |
| WebXR / WebSpatial           | Web permission / UA policy |             Web permission / UA policy |   Web permission / UA policy |
| IndexedDB / Cache API / OPFS |             Origin storage |         Origin storage / open question |   Origin storage / UA policy |
| Persistent storage           |                  UA policy |                          Open question | UA policy, possibly stronger |
| Service worker               |                  UA policy |                              UA policy |                    UA policy |
| Background Sync              |                  UA policy |                          Open question |                    UA policy |
| Periodic Background Sync     |                 Restricted |                   Likely install-gated |                    UA policy |
| Push / notifications         |     Permission + UA policy | Permission + UA policy / open question |       Permission + UA policy |
| Badging                      |                    Limited |                              Likely no |            Yes, if supported |
| File handlers                |                         No |                                     No |            Yes, if supported |
| Protocol handlers            |        Limited / UA policy |                          No or limited |            Yes, if supported |
| URL handlers                 |                         No |                                     No |            Yes, if supported |
| Share target                 |                         No |                                     No |            Yes, if supported |
| Shortcuts                    |                         No |                                     No |            Yes, if supported |

This table is not meant to be final. The main point is that windowing and install-gated OS integration should be separable.

## Relationship to install

Install is still useful and should still exist.

The difference is that install should mean “I want to keep this app and let it integrate more deeply with the system,” not “I need to do this just to see the app in the right kind of window.”

With this proposal:

* first launch can be link-based and low-friction;
* standalone presentation can happen before install;
* install remains the step for durable identity and deeper integration.

## Why use the manifest?

Standalone mode removes or reduces normal browser chrome. That has spoofing and trust implications.

So this should not be something any page can do accidentally. The app should have to opt in through the manifest, and the UA should still enforce the usual requirements:

* HTTPS;
* valid manifest;
* scope checks;
* display mode;
* app name/icons/identity metadata;
* installability checks, if the UA requires them;
* user activation or explicit user choice, if the UA requires it.

The manifest opt-in does not grant new permissions. It only says that the app would like to be eligible for standalone launch before install.

## UA affordance

If the browser URL bar goes away, the UA still needs to give the user some way to understand and control what is happening.

The exact UI can be platform-specific, but it probably needs to support:

* viewing the current origin or URL;
* opening the page back in the browser;
* installing the app;
* closing or managing the standalone session;
* viewing permissions or site settings.

For a spatial browser, this might be part of the spatial shell instead of normal browser chrome.

## Storage

Storage is one of the main open questions.

Possible models:

1. **Use normal origin storage**
   The standalone session shares storage with the same origin in the browser. This is probably the simplest web-shaped model.

2. **Use temporary storage**
   The standalone session gets storage that may be cleared when the session ends. This might match a “preview” model, but could surprise developers.

3. **Use app-like storage and promote it on install**
   The standalone session gets separate app storage that later becomes installed-app storage. This may be harder to standardize and more platform-specific.

My current assumption is that shared origin storage is the cleanest default, with persistence still controlled by existing browser policy. Install may affect persistence heuristics, quota, or eviction policy, but install-free standalone should not silently create a durable installed-app storage model.

Open questions:

* Does install-free standalone share browser-origin storage?
* Should install make the same storage more durable?
* If storage is temporary, how does the site know?
* If the user later installs, what state is preserved?

## Security and privacy notes

A few things need care:

* **Spoofing:** if browser chrome is removed, the UA should still expose origin/URL somehow.
* **User intent:** standalone-before-install should not look like the user installed the app.
* **Capability escalation:** install-free mode should not unlock install-only capabilities.
* **Cleanup/revocation:** if the UA remembers a standalone preference, users should be able to undo it.
* **Permissions:** camera, mic, geolocation, XR, spatial APIs, files, clipboard, etc. should keep using the normal web permission model.

## Spatial motivation

This matters more for spatial web apps than for many 2D apps.

In a spatial environment, the difference between “browser tab” and “standalone app window” is not just cosmetic. The standalone surface may define how the app is placed, resized, focused, moved, or rendered in space.

A spatial app may need:

* an independent spatial panel;
* a transparent or borderless surface;
* multiple windows;
* a volume-like presentation;
* app lifecycle controlled by the spatial shell;
* WebSpatial or WebXR behavior that does not make sense inside a normal browser tab.

If install is required before any of that is visible, users are asked to commit too early.


## Possible names

`install_free` is probably not the final name. Other possible names:

* `launch_without_install`
* `standalone_before_install`
* `preview_standalone`
* `allow_standalone_launch`

The name should make clear that this is about launch/presentation before install, not bypassing install-gated capabilities.

## Feature detection

Today, apps can detect display mode with media queries:

```js
window.matchMedia('(display-mode: browser)').matches
window.matchMedia('(display-mode: standalone)').matches
window.matchMedia('(display-mode: minimal-ui)').matches
```

That may be enough for some cases, but it does not say whether the app is formally installed or just running install-free standalone.

Open questions:

* Should the app be able to tell install-free standalone from installed standalone?
* Should this be exposed in CSS, JS, both, or neither?
* Would exposing it help with install prompts?
* Would exposing it encourage sites to give users a worse experience before install?

## Open questions

### Launch

* Should first launch require a user action?
* Should the browser show an “Open as standalone app” button?
* Can a normal link open directly into install-free standalone mode?
* Can the UA remember the user’s choice?
* Is that remembered by origin, manifest id, scope, or device profile?

### Manifest requirements

* Is `display: standalone` required?
* How does this interact with `display_override`?
* Should the app need to pass existing installability checks?
* Which manifest fields are required?

### Storage

* Is storage shared with the browser tab?
* Is storage temporary until install?
* Does install promote storage durability?
* What happens when the standalone window is closed?
* What happens if the user later installs?

### Capability gating

* Which capabilities are install-only?
* Which ones are just normal web permissions?
* What happens with push, notifications, badging, file handlers, protocol handlers, URL handlers, share target, shortcuts, Background Sync, Periodic Background Sync, and persistent storage?

### UI / security

* How does the user see the current origin?
* How does the user return to the browser?
* How is install promoted?
* Should install-free standalone look different from installed app?
* How does the user clear a remembered standalone-launch choice?

### Spatial

* How does this work for spatial windows, panels, and volumes?
* Does each in-scope navigation create a new spatial container?
* What is the lifecycle when a spatial window closes?
* How is install shown inside an XR shell?
* Can install-free apps use transparent or borderless spatial surfaces?

## Prior work

- [Web App Manifest](https://www.w3.org/TR/appmanifest/)
- [Google Play Instant Apps](https://developer.android.com/topic/google-play-instant) (deprecated; same UX goal on Android)
- [Apple App Clips](https://developer.apple.com/app-clips/)
- [Window Controls Overlay](https://wicg.github.io/window-controls-overlay/)
- Alex Russell, ["Progressive Apps: Escaping Tabs Without Losing Our Soul"](https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/) (2015)
- [WICG](https://wicg.io/)


## Conclusion

Install currently bundles together too many things: standalone presentation, durable identity, OS integration, background capabilities, and long-term retention.

For spatial and app-like web apps, we should be able to separate these.

The proposal is:

> Let an eligible web app open in standalone mode before install. Keep install-only capabilities install-only.

That keeps the web’s low-friction link launch model, while still allowing richer app and spatial experiences.

