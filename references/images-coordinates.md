# Images, Coordinates, And Pixel Buffers Reference

Use this reference when debugging wrong pixels, upside-down images, crashes during render, proxy issues, temporal mapping, or tile/full-frame behavior.

## Spatial Coordinates

OpenFX requirements:

- Positive X points right and positive Y points up.
- Canonical coordinates describe the ideal image plane with double precision.
- Pixel coordinates describe addressable pixels with integer coordinates.
- RoD and RoI actions use canonical coordinates.
- Image bounds and render windows use pixel coordinates.
- Pixel aspect ratio and render scale are needed when mapping between canonical and pixel coordinates.

Recommendation:

- Be explicit in code comments and variable names: `OfxRectD` for canonical rectangles, `OfxRectI` for pixel rectangles.
- Do not mix render-window pixel coordinates with RoI/RoD canonical coordinates without conversion.

Official source:

- Coordinate Systems: https://openfx.readthedocs.io/en/main/Reference/ofxCoordSystem.html

## Temporal Coordinates

OpenFX requirements:

- OFX Image Effect time is output-frame time referenced to the start of the effect.
- UI frame labels are not the same as OFX effect time.
- Clip frame ranges and clip FPS describe how clip images map into the effect time domain.
- Clips with frame rate `0` may be continuously samplable.

Recommendation:

- For frame mapping bugs, log output time, output range, source range, source FPS, output FPS, and computed source time.
- Avoid parity logic based on displayed start frames like `1001`; use effect-relative time/range.

## Image Buffers

OpenFX requirements:

- OFX images are rectangular arrays of pixels.
- Supported component layouts include RGBA, RGB, and alpha images.
- Supported component depths include byte, short, half-float, and float values as defined by the active OpenFX headers and host.
- RGBA pixels are packed as `R, G, B, A`.
- Images are left-to-right and bottom-to-top, with the data pointer at the bottom-left pixel.
- Scanlines need not be tightly packed.
- `rowBytes` is the byte stride between rows and may be negative.
- Bounds are inclusive on left/bottom and exclusive on right/top: `x1 <= x < x2`, `y1 <= y < y2`.
- Image bounds may be larger, smaller, or equal to the image RoD depending on architecture and tiling.
- A fetched image must be released in the same action.

Recommendation:

- Never compute row start as `y * width * components` unless a local packed buffer was created by your own code.
- Use bounds-aware access helpers.
- Log bounds, RoD, rowBytes, component/depth, and render scale when debugging pixel issues.
- If an external runtime expects top-left-first rows, flip Y while packing and flip back while unpacking.

Minimal row access pattern:

```cpp
auto* row = reinterpret_cast<unsigned char*>(base)
          + (y - bounds.y1) * rowBytes;
auto* px = reinterpret_cast<float*>(row)
         + (x - bounds.x1) * components;
```

Official source:

- Images and Clips: https://openfx.readthedocs.io/en/main/Reference/ofxImageClip.html

## Render Window And Tiling

OpenFX requirements:

- The host may ask an effect to render a window of pixels.
- The requested window is passed through `kOfxImageEffectPropRenderWindow`.
- If rendering outside RoD, fill those pixels with transparent black.
- Multiple render actions may be passed at the same time according to declared thread-safety.

Recommendation:

- If the algorithm is local and supports tiles, process only the render window and avoid reading/writing outside it.
- If the algorithm requires full-frame data, advertise no tile support, but still validate actual bounds and render windows.
- Test proxy, thumbnail, full-res render, and tiled/full-frame hosts separately.

Official source:

- Rendering: https://openfx.readthedocs.io/en/main/Reference/ofxRendering.html

## Threading And Sequential Rendering

OpenFX requirements:

- A plugin declares render thread-safety through `kOfxImageEffectPluginRenderThreadSafety`.
- `RenderUnsafe`: only a single render action at a time among all instances.
- `RenderInstanceSafe`: one render action at a time per instance.
- `RenderFullySafe`: multiple simultaneous renders may occur on the same instance.
- If fully safe and host frame threading is enabled, the host may render different windows of the same frame simultaneously.
- Sequential-render properties exist for effects that need or prefer in-order frame rendering, but hosts may not always guarantee it, especially interactively.

Recommendation:

- If a third-party SDK context is not thread-safe, guard it with a mutex and avoid claiming full safety for the instance.
- Do not make correctness depend on render calls arriving in frame order unless the plugin explicitly declares and verifies sequential rendering behavior.

Official source:

- Thread and Recursion Safety: https://openfx.readthedocs.io/en/main/Reference/ofxThreadSafety.html
- Rendering: https://openfx.readthedocs.io/en/main/Reference/ofxRendering.html

## Fields, PAR, And Proxy Render Scale

OpenFX requirements:

- Images/clips may be fielded or full-frame.
- The Y axis points up in OFX field terminology.
- Render scale and pixel aspect ratio affect coordinate mapping.
- Clip preferences can be used to request fielding, PAR, and other output properties where the context allows it.

Recommendation:

- If the plugin is not designed for interlaced/fielded footage, document and test the unsupported path.
- For ML/full-frame algorithms, prefer full-frame progressive input unless the product explicitly supports fielded media.
- Include render scale and field in cache keys if they change runtime input pixels.
