---
name: openfx-plugin
description: Build, debug, review, or document host-agnostic OpenFX Image Effect plugins. Use when working on .ofx plugins, OpenFX C/C++ Support Library code, OFX bundle packaging, image-effect contexts such as Filter/General/Generator/Transition/Paint/Retimer, temporal frame access, clips, parameters, render actions, pixel buffers, rowBytes, tiling, thread-safety, status codes, or integration with hosts such as DaVinci Resolve, Nuke, Natron, Baselight, or other OFX hosts.
---

# OpenFX Plugin

Use this skill for OpenFX Image Effect plugin work: interface design, code review, debugging, packaging, host integration, and documentation. Keep the OpenFX contract separate from project-specific SDKs, models, products, or host folklore unless the user explicitly asks for a host-specific extension.

This skill is intentionally **Image Effect API only**. It does not cover OpenFX Mesh Effect/OpenMfx, custom host SDKs, or non-OFX scripting workflows except when comparing extension options.

## Source Authority

Treat only the OpenFX specification, official OpenFX headers, and official OpenFX examples as normative requirements. Everything else in this skill is guidance unless verified against the target host/runtime.

Use this skill as an operating guide that combines stable OpenFX engineering patterns, project-specific context, and official-source verification. For simple, stable workflow logic already captured here, it is acceptable to follow the skill directly. For any question, diagnosis, implementation, or code review that depends on what OpenFX requires, first check the relevant official OpenFX page, official header, or official example before presenting it as a requirement.

Decision order:

1. Official OpenFX docs/headers/examples define OpenFX requirements.
2. The current project code, build files, package contents, logs, and tests define what this project actually does.
3. Verified target-host documentation/logs/runtime behavior define host-specific requirements.
4. This skill provides reusable workflow, review logic, and engineering recommendations.

If official sources and this skill disagree, follow the official source and update or qualify the skill. If project evidence and this skill disagree, inspect the project state and treat the skill as stale guidance until updated. If the official source is unavailable during the turn, say that the answer is based on local/offline references and should be verified against the official documentation before treating it as normative.

Use these labels when explaining or changing code:

- `OpenFX requirement`: directly required by official OpenFX docs, headers, or examples.
- `Host-specific requirement`: required by a verified target host, version, or SDK.
- `Recommendation`: engineering practice that is usually safer but not mandated by OpenFX.
- `Project policy`: a local team or product decision.

Do not reject a working implementation solely because it differs from a recommendation here. Reject it only if it violates official OpenFX, verified target-host behavior, or an explicit project policy.

## Progressive Reference Router

Read only the references needed for the task:

- For plugin contract, exported symbols, suites, objects, contexts, temporal coordinates, or official source links, read `references/official-contract.md`.
- For `describe`, `describeInContext`, `createInstance`, `render`, `isIdentity`, RoD/RoI, `getFramesNeeded`, clip preferences, changed actions, purge caches, and status codes, read `references/lifecycle-actions.md`.
- For user parameters, animated values, choice compatibility, pages/groups, push buttons, private persisted state, and parameter UI layout, read `references/parameters-ui.md`.
- For pixel buffers, bounds, rowBytes, coordinate systems, render windows, tiling, fields, proxies, and format conversion, read `references/images-coordinates.md`.
- For property sets, OpenFX suites, image memory allocation, progress/message/timeline helpers, and custom Interact/overlay UI, read `references/suites-interacts.md`.
- For bundle layout, platform packaging, runtime dylibs, `install_name`/`rpath`, license notices, and clean-machine distribution, read `references/packaging-hosts.md`.
- For C++ Support Library patterns and official example style, read `references/support-library-patterns.md`.

After reading a reference file, combine it with the current project state. Verify any normative claim against the linked official OpenFX page, the official headers in the checked-out OpenFX SDK, or an official OpenFX example before presenting it as an `OpenFX requirement`.

If the user asks for a broad audit, read all references listed above, then sample-check the relevant official sources for each risk area. If the user asks for host-specific behavior, start with the official OpenFX reference, then verify the host documentation/logs separately.

## Task Workflow

1. Identify the target host(s), platform(s), and whether the work is prototype, production, or commercial distribution.
2. Identify the OpenFX context(s): `Filter`, `General`, `Generator`, `Transition`, `Paint`, or `Retimer`.
3. For every normative OpenFX claim, inspect the relevant official page/header/example before deciding or answering.
4. List the contract: clips, parameters, components, bit depths, temporal access, tiling, thread-safety, RoD/RoI, clip preferences, parameter persistence, and packaging.
5. Mark each statement as `OpenFX requirement`, `Host-specific requirement`, `Recommendation`, or `Project policy`.
6. Implement or review against the narrowest official contract first.
7. Verify with build output, host logs, render tests, and package inspection before calling the work complete.

## Contract Checklist

Before editing code, answer these:

- Which contexts are advertised, and are all mandated clips/parameters defined for each one?
- Are parameter names, defaults, animation behavior, choice ordering, persistence, and UI grouping compatible with saved projects?
- Which components and bit depths are advertised, and does render validate what the host delivered?
- Does the plugin need random temporal access? If yes, does the effect and the relevant clips declare it, and does `getFramesNeeded()` match `render()`?
- Does the algorithm support tiles, or must it request full-frame behavior and still respect render windows/bounds safely?
- What render thread-safety level is declared, and is mutable state protected accordingly?
- Are output identity/no-op paths exact enough for `isIdentity()`?
- Are RoD, RoI, frame-varying, continuous-sample, fielding, PAR, and output frame-rate preferences handled intentionally?
- Are runtime dependencies packaged bundle-locally or otherwise resolved on a clean machine?

## Implementation Defaults

Use these defaults only when they fit the project:

- Prefer the OpenFX C++ Support Library when the repository already uses it; use raw C when the project intentionally avoids the support layer.
- Keep OFX host communication in the plugin layer and put external SDK/GPU/model calls behind a small wrapper.
- Keep instance-owned caches and runtime contexts instance-local unless sharing has been proven safe.
- Pick thread-safety conservatively. If one instance owns mutable caches or a non-thread-safe SDK context, serialize access or advertise a less aggressive safety level.
- Disable tiles for full-frame algorithms such as optical flow, global transforms, or ML inference that cannot produce correct results from partial windows alone.
- Use `isIdentity()` for exact disabled/pass-through/no-op states, not for approximate fallbacks.
- Include cache keys that cover time, size, source identity/content, relevant parameters, and runtime configuration.
- Prefer deterministic fallback plus logging over host crashes when an external runtime fails, unless product policy requires a hard error.

## Debugging Order

When an OFX plugin misbehaves, check in this order:

1. Bundle is discovered, loaded, and exports the expected OFX entry points.
2. Host enumerates the expected plugin identifier and version.
3. `describe()` and `describeInContext()` run for the expected context.
4. `createInstance()` fetches all required clips and parameters.
5. Descriptor declarations match the code path: context, clips, components, bit depth, temporal access, tiling, thread-safety.
6. `getFramesNeeded()` and `render()` request the same source times.
7. `isIdentity()`, RoD/RoI, and clip preferences do not accidentally bypass render or cause wrong host caching.
8. Pixel conversion handles bounds offsets, bottom-left origin, rowBytes, component order, bit depth, render scale, and fields.
9. Render code tolerates out-of-order, repeated, proxy, thumbnail, and parallel calls.
10. External runtime calls are serialized if needed and errors are converted to OFX-safe status/fallback behavior.
11. Package resolves all dylibs/resources/licenses on a clean target system.

## Validation Gate

For production or commercial work, do not stop at a successful local preview. Validate:

- Clean build from a fresh build directory.
- Bundle structure and binary dependencies.
- Host load logs and context selection.
- Enabled, disabled, identity, edge-frame, missing-source, proxy, thumbnail, cache, and full-render paths.
- Every advertised context, component, bit depth, and parameter path.
- Temporal access with non-sequential frame requests if temporal access is advertised.
- Unsupported formats fail safely instead of being read incorrectly.
- Third-party license notices are present for redistributed code, support libraries, model files, icons, and runtime binaries.

## Official Links

For normative OpenFX questions, use official sources before relying on memory or on this skill. For simple stable workflow steps, the skill can be followed directly, but project evidence and official sources still override it. When internet access is available, prefer the current online OpenFX docs; when offline, use the official headers and examples in the local OpenFX SDK checkout and state that the source is local/offline:

- OpenFX docs home: https://openfx.readthedocs.io/en/main/
- Generic Core API: https://openfx.readthedocs.io/en/main/Reference/ofxCoreAPI.html
- Image Effect API: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectAPI.html
- Image Effect Contexts: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectContexts.html
- Image Effect Actions: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectActions.html
- Coordinate Systems: https://openfx.readthedocs.io/en/main/Reference/ofxCoordSystem.html
- Images and Clips: https://openfx.readthedocs.io/en/main/Reference/ofxImageClip.html
- Effect Parameters: https://openfx.readthedocs.io/en/main/Reference/ofxParameter.html
- Rendering: https://openfx.readthedocs.io/en/main/Reference/ofxRendering.html
- Interacts: https://openfx.readthedocs.io/en/main/Reference/ofxInteracts.html
- Clip Preferences: https://openfx.readthedocs.io/en/main/Reference/ofxClipPreferences.html
- Suites Reference: https://openfx.readthedocs.io/en/main/Reference/suites/ofxSuiteReference.html
- Packaging: https://openfx.readthedocs.io/en/main/Reference/ofxPackaging.html
- Status Codes: https://openfx.readthedocs.io/en/main/Reference/ofxStatusCodes.html
- ASWF OpenFX repository: https://github.com/AcademySoftwareFoundation/openfx
