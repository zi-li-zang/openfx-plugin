# C++ Support Library And Example Patterns

Use this reference when the project uses the OpenFX C++ Support Library or when translating official raw C concepts into C++ overrides.

## Role Of The Support Library

OpenFX itself is a C ABI and suite/property contract. The C++ Support Library is a helper layer used by many examples and plugins; it is not the OpenFX protocol itself.

Recommendation:

- Use the Support Library when the repository already uses it because it reduces manual action-dispatch and property boilerplate.
- Do not assume every host or plugin uses the same wrapper version.
- When a wrapper method is unclear, check the wrapper headers and the official raw C action docs.

## Common Override Mapping

Typical C++ Support Library mapping:

- Global describe action -> factory `describe()`.
- Describe-in-context action -> factory `describeInContext()`.
- Create instance -> plugin class constructor.
- Destroy instance -> plugin destructor.
- Render action -> `render(const OFX::RenderArguments&)`.
- Is identity action -> `isIdentity(...)`.
- Get region of definition -> `getRegionOfDefinition(...)`.
- Get regions of interest -> `getRegionsOfInterest(...)`.
- Get frames needed -> `getFramesNeeded(...)`.
- Get clip preferences -> `getClipPreferences(...)`.
- Instance changed -> `changedParam(...)` / `changedClip(...)`.

Recommendation:

- Keep raw C action names in comments or docs for engineers comparing wrapper code to official docs.
- Verify purge-cache handling in the specific wrapper version; not every wrapper exposes it through the same virtual method.

## Minimal Descriptor Shape

```cpp
void describe(OFX::ImageEffectDescriptor& desc) {
    desc.setLabels("Display Name", "Display Name", "Display Name");
    desc.setPluginGrouping("Vendor");
    desc.addSupportedContext(OFX::eContextFilter);
    desc.addSupportedBitDepth(OFX::eBitDepthFloat);
    desc.setSupportsTiles(false);
    desc.setRenderThreadSafety(OFX::eRenderInstanceSafe);
}
```

OpenFX requirement:

- Only advertise capabilities that render and related actions can satisfy.

Recommendation:

- Treat descriptor code as the host contract. Keep it short, obvious, and audited.

## Minimal Context Shape

```cpp
void describeInContext(OFX::ImageEffectDescriptor& desc,
                       OFX::ContextEnum context) {
    OFX::ClipDescriptor* src =
        desc.defineClip(kOfxImageEffectSimpleSourceClipName);
    src->addSupportedComponent(OFX::ePixelComponentRGBA);
    src->setSupportsTiles(false);

    OFX::ClipDescriptor* dst =
        desc.defineClip(kOfxImageEffectOutputClipName);
    dst->addSupportedComponent(OFX::ePixelComponentRGBA);
    dst->setSupportsTiles(false);

    OFX::BooleanParamDescriptor* enabled =
        desc.defineBooleanParam("enabled");
    enabled->setDefault(true);
}
```

OpenFX requirement:

- Define mandated context clips and parameters during describe-in-context.

Recommendation:

- Avoid branching that makes one context accidentally miss a mandated clip.

## Minimal Instance Shape

```cpp
class MyPlugin : public OFX::ImageEffect {
public:
    explicit MyPlugin(OfxImageEffectHandle handle)
        : OFX::ImageEffect(handle) {
        dstClip_ = fetchClip(kOfxImageEffectOutputClipName);
        srcClip_ = fetchClip(kOfxImageEffectSimpleSourceClipName);
        enabled_ = fetchBooleanParam("enabled");
    }

    void render(const OFX::RenderArguments& args) override;

private:
    OFX::Clip* dstClip_ = nullptr;
    OFX::Clip* srcClip_ = nullptr;
    OFX::BooleanParam* enabled_ = nullptr;
};
```

Recommendation:

- Store fetched handles and per-instance runtime state on the instance object.
- Use RAII image wrappers where available so host-fetched images are released on all exits.

## Identity Pattern

```cpp
bool isIdentity(const OFX::IsIdentityArguments& args,
                OFX::Clip*& identityClip,
                double& identityTime) override {
    if (!enabled_->getValueAtTime(args.time)) {
        identityClip = srcClip_;
        identityTime = args.time;
        return true;
    }
    return false;
}
```

OpenFX requirement:

- Return identity only for exact input passthrough at a specific time.

## Frames Needed Pattern

```cpp
void getFramesNeeded(const OFX::FramesNeededArguments& args,
                     OFX::FramesNeededSetter& setter) override {
    OfxRangeD range;
    range.min = std::floor(args.time);
    range.max = range.min + 1.0;
    setter.setFramesNeeded(*srcClip_, range);
}
```

OpenFX requirement:

- The requested ranges must cover what render will fetch.

Recommendation:

- For algorithms with conditional paths, make the range calculation share logic with render-time source mapping.

## Example-Inspired Review Habits

Official examples commonly demonstrate:

- Returning source RoD for same-size filters.
- Setting source RoI from the requested output region.
- Using identity for no-op states.
- Updating UI/enabled state in changed actions.
- Marking generators/noise as frame-varying where needed.

Recommendation:

- Treat examples as patterns, not proof that every production plugin should copy the same defaults.
- When an example omits a production concern such as packaging, logging, or runtime errors, add those as project engineering layers rather than calling them OpenFX requirements.
