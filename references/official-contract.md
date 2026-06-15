# Official Contract Reference

Use this reference when the task needs the official OpenFX contract: exported entry points, plugin identity, suites, image-effect objects, contexts, temporal coordinates, or official source links.

## Authority Boundary

OpenFX requirements should come from:

- Official OpenFX specification pages.
- Official OpenFX headers such as `ofxCore.h`, `ofxImageEffect.h`, `ofxProperty.h`, `ofxParam.h`, and related headers.
- Official OpenFX examples in the ASWF/OpenFX repository.

Everything else is a recommendation, host-specific observation, or project policy until verified.

## Core Loading Contract

OpenFX is a C ABI contract between host and plugin. A plugin binary exposes C entry points and returns `OfxPlugin` records. The host then communicates through function pointers and property/suite handles.

OpenFX requirements:

- A plugin binary implements and exports `OfxGetNumberOfPlugins()`.
- A plugin binary implements and exports `OfxGetPlugin(int nth)`.
- Current OpenFX documentation also defines an optional exported `OfxSetHost(const OfxHost *host)` function, added in 2020. Hosts must check whether the symbol exists and must not assume older plugins implement it.
- `OfxSetHost` may return `kOfxStatFailed` to tell the host the binary has no plugins to expose for that host and should be skipped silently.
- The host pointer passed to `OfxSetHost` is only temporary for the early exposure decision; normal plugin/host communication still uses the host pointer later passed through `OfxPlugin::setHost`.
- Each returned `OfxPlugin` has `pluginApi`, `apiVersion`, `pluginIdentifier`, major/minor version fields, `setHost`, and `mainEntry`.
- Image Effect plugins set `pluginApi` to `kOfxImageEffectPluginApi`.
- The host calls the plugin `mainEntry` with action strings, a handle, input properties, and output properties.
- During loading, the host loads the binary, optionally calls `OfxSetHost` if implemented, calls `OfxGetNumberOfPlugins()`, calls `OfxGetPlugin()` for each index, checks `pluginApi` and `apiVersion`, ignores unsupported plugins, records supported plugin pointers, and passes an `OfxHost` through the `setHost` function pointer in `OfxPlugin`.
- `OfxPlugin::setHost` is mandatory and must only store the host pointer; the plugin must not call OFX suite functions from inside `setHost`.

Recommendation:

- Keep exported OFX symbols stable and visible even if the rest of the binary uses hidden symbols.
- Treat `OfxSetHost` as a compatibility-sensitive current-API hook: useful for host-dependent plugin exposure, but not safe to assume in older hosts/loaders.
- Use reverse-domain plugin identifiers for uniqueness.
- Treat major version bumps as compatibility-breaking; minor version bumps should preserve existing setups.

Official source:

- Generic Core API: https://openfx.readthedocs.io/en/main/Reference/ofxCoreAPI.html
- `OfxPlugin` struct: https://openfx.readthedocs.io/en/main/Reference/ofxPluginStruct.html
- Image Effect API: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectAPI.html
- Loading sequence: https://openfx.readthedocs.io/en/main/Reference/ofxCoreAPI.html

## Host Communication Model

OpenFX suites are C structs of function pointers fetched from the host. This avoids linking directly against host symbols.

OpenFX requirements:

- Fetch required suites from `OfxHost::fetchSuite()`.
- Check suite availability/version before using optional host features.

Recommendation:

- Return `kOfxStatErrMissingHostFeature` or another action-appropriate status when a required host feature is absent.
- Copy host-returned strings before retaining them beyond the documented lifetime for the relevant suite/action.
- Log host name/version and suite availability in diagnostic builds.
- Keep suite pointers in a small bootstrap layer or use the C++ Support Library if already adopted.

## Image Effect Objects

Official object model:

- Host Descriptor: describes host capabilities.
- Image Effect Descriptor: plugin description cached by the host.
- Image Effect Instance: live instance between create and destroy.
- Clip Descriptor/Instance: input or output image sequence definition and live handle.
- Parameter Descriptor/Instance: user-visible or mandated host-controlled state.
- Image Instance: fetched image data; must be released during the same action.
- Interact Descriptor/Instance: custom UI/overlay surfaces, separate from normal render.

OpenFX requirements:

- Define clips and parameters during descriptor actions, not dynamically during render.
- Fetch live clip/parameter handles from an effect instance after creation.
- Release fetched images before returning from the action.
- Do not retain host image handles beyond the action that fetched them.

Official source:

- Image Effect API main objects: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectAPI.html
- Images and Clips: https://openfx.readthedocs.io/en/main/Reference/ofxImageClip.html

## Contexts

OpenFX Image Effect contexts define mandated clips, parameters, and behavior constraints.

OpenFX requirements:

- All contexts have one output clip and zero or more input clips.
- A host or plugin does not have to support all contexts.
- A plugin declares supported contexts on the effect descriptor.
- The host calls `DescribeInContext` for each supported context it wants described.
- Mandated context clips/parameters must be defined in `DescribeInContext`.

Common contexts:

- `Generator`: output clip only; no compulsory input.
- `Filter`: one compulsory input clip and output clip.
- `Transition`: two compulsory input clips, output clip, and host-controlled `Transition` double parameter.
- `Paint`: source/paint/mask-style inputs according to the context contract.
- `Retimer`: input `Source`, output `Output`, and host-controlled `SourceTime` double parameter.
- `General`: output clip and arbitrary input design; often used in tree compositors.

Retimer-specific requirements:

- The `SourceTime` parameter is host-controlled; the plugin reads it and must not label, position, or expose it as a normal UI control.
- The retimer output at a time is based on the source time returned by `SourceTime` at that output time.
- Source and output component/depth preferences are constrained in retimer context.

Recommendation:

- Do not implement `Retimer` unless the plugin is actually intended to be selected as a host retiming algorithm.
- For normal node/filter effects, do not assume retimer `SourceTime` semantics.

Official source:

- Contexts: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectContexts.html

## Temporal Coordinates

OpenFX effect time is not necessarily the same concept as a host's displayed frame label such as `1001`. Time and range interpretation should be derived from the host-provided OFX properties, clip frame ranges, and clip frame rates.

OpenFX requirements:

- Times passed through the Image Effect API use the OFX temporal coordinate system.
- Clip instances expose frame ranges and frame rates.
- Some clips may be continuously samplable and report frame rate `0`.

Recommendation:

- To map an output time to input time when source/output frame rates differ, use the clip range and FPS information provided by the host.
- Base phase calculations on OFX effect time/range, not on UI frame labels.
- Log output time, source range, output range, source FPS, output FPS, and computed source time when debugging retiming or temporal access.

Official source:

- Coordinate Systems: https://openfx.readthedocs.io/en/main/Reference/ofxCoordSystem.html
- Images and Clips: https://openfx.readthedocs.io/en/main/Reference/ofxImageClip.html
