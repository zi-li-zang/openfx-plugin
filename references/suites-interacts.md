# Suites, Memory, Messages, And Interacts Reference

Use this reference when a plugin needs host suites beyond basic render, temporary image memory, progress/cancel UI, user messages, timeline access, or custom overlay/parameter interact UI.

## Suite Negotiation

OpenFX requirements:

- The host exposes capabilities through suites: property, image effect, memory, parameter, multithreading, interact, message, timeline, progress, and others.
- A plugin obtains suites through the host's `fetchSuite` callback.
- Suite availability and versions depend on host support.

Recommendation:

- If a required suite or version is unavailable, return an action-appropriate missing-feature/unsupported status rather than calling through a null or wrong-version pointer.
- Fetch required suites during load/describe/bootstrap and fail early when a hard dependency is missing.
- Treat optional suites as feature flags and keep graceful fallbacks.
- Log host name/version and suite availability in diagnostic builds.

Official source:

- Image Effect API suites overview: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectAPI.html
- Suites Reference: https://openfx.readthedocs.io/en/main/Reference/suites/ofxSuiteReference.html

## Property Sets

OpenFX requirements:

- OFX objects expose state through property sets.
- Properties are typed and may be single-valued or multi-dimensional arrays.
- Property names are strings defined by the API headers or by documented per-action naming patterns.
- The Property Suite is used to get/set property values, query dimensions, and reset/delete values where supported.

Recommendation:

- Copy string values returned by host suites if plugin code needs to retain them beyond the relevant call/action.
- Treat property dimensions as part of the contract; do not assume a property has exactly one value unless the official definition says so.
- Log property dimensions and values when debugging mismatched host behavior.
- In raw C plugins, centralize property access helpers to avoid index/type mistakes.

## Image Memory

OpenFX requirements:

- The Image Effect Suite exposes image memory allocation functions for memory associated with an image effect instance.
- Memory allocated with `imageMemoryAlloc` must be released with `imageMemoryFree`.
- To access the allocated memory, lock the memory handle with `imageMemoryLock` and unlock it with `imageMemoryUnlock`.

Recommendation:

- Use RAII wrappers for OFX image-memory handles.
- Prefer ordinary process memory for purely local CPU scratch buffers unless host-managed image memory is needed or expected by the host/runtime.
- Do not confuse image-memory handles with fetched image handles; they have different lifecycles.

Official source:

- Auto-generated reference index: https://openfx.readthedocs.io/en/main/Reference/DoxygenIndex.html

## Messages And Progress

OpenFX requirements:

- Message and progress suite availability depends on host support.
- Message suite calls are host-mediated user messages.
- Progress suite support is intended for longer operations where progress/abort feedback is useful.

Recommendation:

- Do not treat host messages as a replacement for diagnostic logs.
- Use message dialogs sparingly and only for actionable user-facing issues.
- For long renders or initialization, use progress and abort checks when the target host supports them.
- Keep file logs for support/debug details that should not interrupt the user.

## Timeline Suite

OpenFX requirements:

- Timeline suite availability is host-dependent.

Recommendation:

- Timeline manipulation is separate from ordinary render-time image fetching and should not be assumed available in all hosts.
- Do not use the timeline suite to compensate for unclear image-effect frame mapping unless the host behavior is verified and the product explicitly requires timeline manipulation.
- Keep timeline-side automation separate from image-effect rendering logic where possible.

## Interacts And Overlay UI

OpenFX requirements:

- Interacts have separate entry points from an image effect's main entry point.
- An interact can receive describe, create, destroy, draw, pen, key, gain-focus, and lose-focus actions.
- Draw actions are where OpenGL drawing is performed.
- Interact pen coordinates are supplied in canonical coordinates; viewport coordinates have origin at the bottom-left.
- Custom parameter interacts and image-effect overlay interacts are host-capability-dependent.

Recommendation:

- Follow the official interact action guidance: keep OpenGL drawing in draw actions rather than pen/key/focus actions.
- Avoid custom interacts unless the UX genuinely requires viewport/parameter drawing.
- Keep render output independent of whether an editor/interactive UI is currently open.
- Test foreground GUI host, background/render-only host, and multiple open editors if using interact state.

Official source:

- Interacts: https://openfx.readthedocs.io/en/main/Reference/ofxInteracts.html
- Actions Passed to an Interact: https://openfx.readthedocs.io/en/main/Reference/ofxInteractActions.html
