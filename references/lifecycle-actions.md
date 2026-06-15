# Lifecycle And Actions Reference

Use this reference for the OpenFX Image Effect action lifecycle and C++ Support Library override mapping.

## Main Action Flow

OpenFX actions:

- The host drives a plugin by calling the plugin's `mainEntry` action dispatcher.
- Common actions include load/unload, describe, describe-in-context, create/destroy instance, begin/end instance changed, instance changed, purge caches, begin sequence render, render, end sequence render, get RoD, get RoI, get frames needed, is identity, get clip preferences, and get time domain.
- An image effect descriptor is used during describe actions; a live image effect instance is used after create instance.

Recommendation:

- In C++ Support Library projects, let the library dispatch actions and override methods such as `describe`, `describeInContext`, `render`, `isIdentity`, `getFramesNeeded`, `getRegionOfDefinition`, `getRegionsOfInterest`, `getClipPreferences`, `changedParam`, and `changedClip`.
- Keep `render()` as a dispatcher and move pixel conversion/runtime calls into small helpers.

Official source:

- Image Effect Actions list: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectAPI.html
- Actions reference: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectActions.html

## Describe

OpenFX requirements:

- Declare supported contexts on the effect descriptor.
- Declare supported bit depths on the plugin during describe; it is an error not to set supported pixel depths.
- Advertise only capabilities the plugin can satisfy.
- Declare render thread-safety according to the implementation.

Recommendation:

- Keep labels, grouping, plugin identifier, bundle metadata, and package name synchronized.
- Use conservative thread-safety until external runtimes and caches are proven safe.
- For full-frame algorithms, set tile support and host-frame-threading policy conservatively.

## Describe In Context

OpenFX requirements:

- Define mandated clips for the selected context.
- Set supported components on input clips; it is an error not to set supported components on input clips during describe-in-context.
- Define mandated pseudo-parameters such as `SourceTime` or `Transition` in the contexts that require them.
- Where the context specifies rules for extra clips, respect those optional/required constraints.

Recommendation:

- Keep parameter names stable because saved host projects may depend on them.
- Avoid adding/removing parameters in minor versions unless defaults preserve old output.

## Create And Destroy Instance

OpenFX requirements:

- Live clip and parameter handles are fetched from the instance after the host has created it.
- A clip instance remains valid while the related effect instance is valid.
- A parameter instance handle remains valid while the associated effect instance remains valid.

Recommendation:

- Store per-instance caches, runtime contexts, mutexes, and configuration in the plugin instance object.
- Delay size-dependent GPU/model initialization until render or begin-sequence-render if dimensions are not known at create time.
- Destroy runtime contexts and caches in the instance destructor/destroy action.

## Render

OpenFX requirements:

- `Render` is passed when the host requires an output frame.
- The host calls begin/end sequence render actions around render sequences before issuing render actions for those sequences.
- Fetch source images only in allowed actions and release them before returning.
- Write only the output/destination image; never modify source clip images.
- Respect the render window and output image bounds. If asked to render outside RoD, fill those pixels with transparent black.
- Do not assume render calls are sequential.

Recommendation:

- Read animated parameters at `args.time`.
- Validate image depth, components, bounds, rowBytes, field, and null handles before reading pixels.
- Keep fallback deterministic and log the real error.
- Poll host abort if long-running render work supports cancellation.

Official source:

- Rendering: https://openfx.readthedocs.io/en/main/Reference/ofxRendering.html
- Images and Clips: https://openfx.readthedocs.io/en/main/Reference/ofxImageClip.html

## Is Identity

OpenFX requirements:

- Use `IsIdentity` only when the output exactly equals one input clip at one time.
- Set both identity clip and identity time when returning identity.
- The default action is to render normally.

Recommendation:

- Good identity cases: disabled effect, zero strength, transition at endpoint, exact passthrough.
- Do not use identity for approximate fallbacks or visually similar results.

Official source:

- Rendering identity effects: https://openfx.readthedocs.io/en/main/Reference/ofxRendering.html
- Is Identity action: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectActions.html

## RoD And RoI

OpenFX requirements:

- `GetRegionOfDefinition` reports how large an image the effect can create.
- `GetRegionsOfInterest` reports how much of each input is needed for a requested output region.
- RoD and RoI action properties are expressed in canonical coordinates.

Recommendation:

- Hosts may render arbitrary windows; verify the exact clipping behavior in the target host.
- Same-size local filters can often use defaults.
- Blur, warp, optical flow, full-frame ML, generators, or image-size-changing effects should reason explicitly about RoD/RoI.
- If tile support is disabled for a full-frame algorithm, still understand RoD/RoI because hosts use it for scheduling and caching.

## Get Frames Needed

OpenFX requirements:

- `GetFramesNeeded` tells the host which input frame ranges are required to compute one output frame.
- If `GetFramesNeeded` is not trapped, the host uses the default frame ranges supplied in the action output properties.
- If temporal access is not enabled, the host assumes only the current input frame is needed and need not call this action.
- Returned frame ranges are continuous pairs and can include multiple discontinuous ranges.
- It is an error to attempt random temporal access if the host does not support it, or if the plugin/clip has not declared temporal access.

Recommendation:

- Keep `getFramesNeeded()` and render-time `fetchImage()` source times exactly aligned.
- Log requested frame ranges during temporal debugging.
- Include temporal inputs in cache keys.

Official source:

- Actions reference: https://openfx.readthedocs.io/en/main/Reference/ofxImageEffectActions.html
- Images and temporal access: https://openfx.readthedocs.io/en/main/Reference/ofxImageClip.html

## Clip Preferences

OpenFX requirements:

- `GetClipPreferences` lets the plugin express preferences for input clips and output clip properties.
- Preference fields include pixel depth, components, pixel aspect ratio, output frame rate, output fielding, output premultiplication, frame-varying behavior, and continuous sampling where supported by host/plugin properties and context.
- Clip preferences cannot animate over the duration of an effect.
- Contexts may restrict which preferences can be changed.

Recommendation:

- Prefer letting the host perform component/depth conversion when possible.
- Set frame-varying when output changes by time even if inputs and parameters appear static.
- Document any output frame-rate or continuous-sample behavior because it affects host caching and timeline mapping.

Official source:

- Clip Preferences: https://openfx.readthedocs.io/en/main/Reference/ofxClipPreferences.html

## Changed Actions And Cache Purge

OpenFX requirements:

- Instance changed actions notify the plugin about parameter or clip changes.
- Purge caches asks the plugin to delete temporary private data caches it can safely release.

Recommendation:

- Clear or version caches when parameters, clips, model paths, runtime modes, or source-format assumptions change.
- Treat begin/end changed actions as batching boundaries when a host changes multiple values.
- Keep cache purge safe: release only rebuildable temporary data, not mandatory persistent state.

## Status Codes

OpenFX requirements:

- OFX actions and suite functions return `OfxStatus`.
- Use `kOfxStatOK` for success.
- `kOfxStatErrMissingHostFeature` is used when the host lacks a required feature and is intended for load/describe/describe-in-context failures.
- `kOfxStatErrUnsupported` indicates an unsupported feature or operation.
- `kOfxStatErrFormat` indicates that something was supplied in the wrong format.
- `kOfxStatErrMemory` indicates a memory shortage.
- `kOfxStatFailed` indicates a failed operation without a more specific status.
- `kOfxStatErrFatal` indicates a likely fatal error.

Recommendation:

- Use `kOfxStatReplyDefault` only when the action has a meaningful default and the plugin intentionally wants it.
- Convert C++ exceptions at the action boundary; do not let arbitrary exceptions cross into the host.
- Prefer `ErrUnsupported` or `ErrFormat` for unimplemented pixel formats rather than reading them incorrectly.
- Use host-facing messages/logs for user-fixable failures such as missing resources.

Official source:

- Status Codes: https://openfx.readthedocs.io/en/main/Reference/ofxStatusCodes.html
