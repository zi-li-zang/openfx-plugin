# Parameters And UI Reference

Use this reference when defining, reviewing, or debugging OFX parameters, animated values, UI grouping, choice compatibility, push buttons, or saved-project compatibility.

## Parameter Ownership

OpenFX requirements:

- OFX parameters are defined and used by the plugin but maintained by the host.
- Define parameters during describe actions. Parameters cannot be defined outside describe actions.
- Parameter names are unique ASCII null-terminated C strings within the plugin and are not necessarily user-visible labels.
- A parameter handle returned during describe is a description handle: set properties on it, but do not get or set live values from it.
- A parameter handle fetched outside describe is an instance handle: get/set values and some mutable properties on it.
- The plugin's persistent state is encoded in its parameter set. If private state must persist, flush it into parameters when appropriate and reconstruct it during create instance.

Recommendation:

- Treat parameter names as saved-project ABI. Rename only with an explicit compatibility plan.
- Keep user-visible labels separate from internal parameter names.
- Avoid storing project-specific runtime state only in memory if users expect saved projects to restore it.

Official source:

- Effect Parameters: https://openfx.readthedocs.io/en/main/Reference/ofxParameter.html

## Parameter Types

OpenFX requirements:

- Official parameter types include integer, double, RGB/RGBA color, boolean, choice, `StrChoice`, string, custom, push button, group, page, and parametric parameters, including multidimensional integer/double variants.
- Multidimensional parameters are treated atomically; all dimensions are set/retrieved together, including keyframes.
- Color parameters are double precision RGB/RGBA values normalized to `[0, 1]`.
- Boolean parameters use integer values `0` or `1`.
- Push button parameters have no value; pressing one sends an instance changed action with user-edited reason.
- Group and page parameters have no values and exist for UI organization.

Recommendation:

- Use the simplest parameter type that matches the user's mental model.
- Prefer `String` file/directory modes for paths when the host UI supports them.
- Use custom parameters only when ordinary parameter types cannot represent the data.

## Animation And Time Sampling

OpenFX requirements:

- Numeric and color parameter types animate by default.
- Group, page, and push button parameters cannot animate.
- Custom, string, boolean, and choice animation depends on host support and host properties.
- `paramGetValue()` returns the current value, while `paramGetValueAtTime()` samples potentially animated values at a specified time.
- If a custom parameter animates, it must provide the custom interpolation callback required by the spec.

Recommendation:

- In render, identity, RoD/RoI, frames-needed, and clip-preference logic, use time-sampled values when animated parameters can affect the result.
- Include relevant animated parameter values or a parameter-version key in render caches.

## Choice And StrChoice Compatibility

OpenFX requirements:

- Choice parameters are stored and returned as integer indices.
- Choice options must not have gaps after describe returns.
- If no default is set, the host should use the first option.
- `kOfxParamPropChoiceOrder` can reorder UI display without changing stored indices when host support exists.
- `StrChoice` parameters store string enums rather than integer indices and are available from OFX 1.5.
- `StrChoice` option and enum arrays must match in length; gaps are errors or undefined behavior.

Recommendation:

- Do not reorder or insert normal choice options in the middle for a minor version unless using and verifying `ChoiceOrder` support.
- For future-compatible choice lists, prefer `StrChoice` when the target hosts support it.
- Check `kOfxParamHostPropSupportsStrChoice` before using `StrChoice` in compatibility-sensitive plugins.
- Avoid removing choice enums used by existing saved projects; if removal is necessary, keep a hidden/deprecated parameter and migrate explicitly.

## Spatial And Time Double Parameters

OpenFX requirements:

- Double parameters can be flagged with semantic types such as plain, angle, scale, time, absolute time, spatial X/Y/XY, and normalized spatial variants.
- Time double parameters are seen by the plugin in frames.
- Spatial double parameters are in canonical coordinates; converting to pixels is the effect's responsibility.

Recommendation:

- Prefer canonical spatial double types over normalized spatial types when supported by the target API/host.
- Document coordinate conversion when parameters drive pixel sampling.

## Pages, Groups, And UI Layout

OpenFX requirements:

- Group parameters create a hierarchy.
- The empty string is the root parent for parameters.
- Page parameters represent paged layouts; `kOfxParamPropPageChild` controls which parameters appear on a page.
- Group parameters cannot be added to a page, and page parameters cannot be added to a page or group.
- Host UI layout is host-dependent.

Recommendation:

- Keep UI grouping simple and host-tolerant.
- Do not assume every host supports the same page layout dimensions, ordering, or custom UI affordances.
- For host-specific polished UI, verify in that host rather than relying on generic OpenFX layout.

## Undo, Changed Actions, And Private Data

OpenFX requirements:

- Use parameter edit begin/end if the plugin changes multiple parameters that should form one undo event.
- Push buttons and parameter edits trigger instance changed actions.
- `SyncPrivateData` tells the plugin to flush private data that needs to persist into the effect parameter set.

Recommendation:

- Hosts usually provide undo/redo for parameter edits, but exact behavior is host-specific.
- Plugin-set parameter changes are normally undoable unless the parameter or edit flow opts out; verify on the target host.
- In changed actions, distinguish user-edited changes from plugin-edited changes to avoid infinite loops.
- Keep button handlers deterministic and undo-aware.
- Use persistent hidden/custom parameters only when needed; do not hide critical product state in transient memory.

## Versioning And Saved Projects

OpenFX requirements:

- Hosts save plugin identifier, major version, and persistent parameters in setups/projects.
- Major version means compatibility-breaking changes.
- Minor version may add parameters if defaults preserve prior behavior.
- It is not an error for a setup to omit a newly added parameter or contain an old parameter no longer used by the plugin.

Recommendation:

- For commercial plugins, write a parameter compatibility plan before changing names, defaults, choices, or persistence.
- Test old project files after every parameter-set change.
