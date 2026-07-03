# Official Docs Index

Use this reference when choosing which official OpenFX source, local snapshot, or topic summary to read. This file is a navigation index, not a replacement for the OpenFX specification, official headers, or official examples.

## Source Order

1. Prefer the current online OpenFX documentation when internet access is available.
2. Prefer official OpenFX headers and official examples from the checked-out OpenFX SDK when working offline or when exact API names, constants, properties, or call signatures matter.
3. Use the local raw snapshot only as an offline copy of the Read the Docs site: `references/sources/openfx-readthedocs-io-en-main.pdf`.
4. Use the topic summaries in `references/*.md` for workflow guidance, recurring review checks, and faster orientation.

If these sources disagree, treat the newest official documentation, official headers, or official examples as authoritative. If relying on the PDF snapshot because the online docs are unavailable, say that the claim is based on a local/offline OpenFX docs snapshot and should be verified against the current official source before treating it as final.

## Raw Snapshot

- File: `references/sources/openfx-readthedocs-io-en-main.pdf`
- SHA-256: `64bac4c30403bec1bcbfae33605cc07997632fd9b6254207acd8383a1accce0e`
- Role: preserved original documentation artifact for offline lookup and source comparison.
- Loading rule: do not load, extract, or quote from the PDF by default. Open it only when a task needs original wording, online docs are unavailable, or the user explicitly asks to use the local snapshot.

When using the snapshot, extract or inspect only the relevant section. Do not paste large blocks of specification text into responses or skill files.

## Topic Router

| Task or question | Read summary first | Primary official pages |
| --- | --- | --- |
| Loading contract, exported symbols, `OfxPlugin`, `OfxSetHost`, host communication, object model, contexts, temporal coordinates | `references/official-contract.md` | Generic Core API, `OfxPlugin` struct, Image Effect API, Image Effect Contexts, Coordinate Systems, Images and Clips |
| Action lifecycle, `describe`, `describeInContext`, instance creation, `render`, `isIdentity`, RoD/RoI, `getFramesNeeded`, clip preferences, changed actions, cache purge, status codes | `references/lifecycle-actions.md` | Image Effect API, Image Effect Actions, Rendering, Images and Clips, Clip Preferences, Status Codes |
| Parameters, animation, choice compatibility, pages/groups, push buttons, private persisted state, parameter UI layout | `references/parameters-ui.md` | Effect Parameters |
| Pixel buffers, bounds, rowBytes, coordinate systems, render windows, tiling, fields, proxies, component/depth conversion | `references/images-coordinates.md` | Coordinate Systems, Images and Clips, Rendering |
| Property sets, suites, image memory allocation, progress/message/timeline helpers, custom Interact/overlay UI | `references/suites-interacts.md` | Suites Reference, Interacts |
| OFX bundle layout, platform packaging, runtime libraries, `install_name`, `rpath`, license notices, clean-machine distribution | `references/packaging-hosts.md` | Packaging |
| C++ Support Library style, factory patterns, official examples, suite bootstrap patterns | `references/support-library-patterns.md` | ASWF OpenFX repository, official examples |

## Online Official Links

- OpenFX docs home: https://openfx.readthedocs.io/en/main/
- Generic Core API: https://openfx.readthedocs.io/en/main/Reference/ofxCoreAPI.html
- `OfxPlugin` struct: https://openfx.readthedocs.io/en/main/Reference/ofxPluginStruct.html
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
