# Packaging And Host Integration Reference

Use this reference for OFX bundle layout, platform differences, runtime dependency placement, commercial distribution, and host-specific verification.

## Official Bundle Model

OpenFX packaging is partly host-specific, but the specification strongly recommends a common bundle hierarchy.

Official packaging facts and recommendations:

- Host search paths, binary format, and directory hierarchy are ultimately host-specific.
- All plugin binaries should end with `.ofx`.
- The recommended package is `NAME.ofx.bundle/Contents/...`.
- macOS uses package-style bundles.
- `Info.plist` is relevant on macOS.
- `Resources` may contain icons, XML/resources, model files, licenses, or runtime assets.
- Architecture subdirectories documented by OpenFX include `MacOS`, `MacOS-x86-64`, `Win32`, `Win64`, `Linux-x86-64`, and others.
- Current OpenFX packaging guidance says macOS plugins should use `Contents/MacOS/`, preferably with a universal binary where that fits the target host matrix. Treat `MacOS-arm64` as host-specific unless the target host documents it.

Typical macOS layout:

```text
Name.ofx.bundle/
└── Contents/
    ├── Info.plist
    ├── MacOS/
    │   └── Name.ofx
    └── Resources/
        ├── optional runtime dylibs
        ├── optional model/data files
        ├── optional icons
        └── license/notice files
```

Official source:

- Packaging: https://openfx.readthedocs.io/en/main/Reference/ofxPackaging.html

## Runtime Dependencies

Recommendation:

- Prefer bundle-local runtime libraries/resources for commercial distribution unless a dependency is intentionally supplied by the host, the operating system, or a documented installer.
- On macOS, inspect dependency paths with tools such as `otool -L`.
- Use `install_name` and `rpath` so the `.ofx` binary resolves the intended bundle-local dylibs instead of developer-machine absolute paths.
- Test on a clean machine/account without source tree, SDK install path, or build directory available.

Review checks:

- No dependency points to the developer workspace.
- No dependency points to an unversioned private build location.
- Model/data paths are relative to bundle resources or user-configurable.
- Missing dependencies fail with a useful message instead of crashing the host.

## Licensing And Notices

License requirements:

- Include notices required by redistributed OpenFX headers/support code and by any other redistributed code, runtime binaries, model weights, examples, icons, fonts, and bundled resources.

Recommendation:

- Add `THIRD_PARTY_NOTICES.txt` or a `Licenses/` folder to the final package.
- Keep license obligations separate for:
  - OpenFX headers/support library.
  - C++ dependencies.
  - ML/runtime SDKs.
  - Model weights.
  - Icons/UI assets.
  - Installer scripts.

## Host-Specific Verification

Recommendation:

- Treat target-host behavior as verified only after checking that host's docs, logs, and actual runtime behavior.
- Hosts may differ in context support, temporal access, thread scheduling, pixel formats, UI surfaces, installation paths, and caching behavior.
- Keep host-specific quirks out of the generic skill unless they are labeled as host-specific and versioned.

Host verification checklist:

- Bundle is discovered at the documented host search path.
- Host logs show the plugin ID and version expected.
- Host selects the intended context.
- Host supplies the advertised component/depth combination.
- Temporal access works with non-sequential frame requests if advertised.
- Proxy/render-cache/final-render paths all produce correct output.

## Commercial Distribution Gate

Before selling or distributing:

- Verify host edition/support matrix for the target users.
- Test a signed/notarized macOS package if distribution requires it.
- Bundle or clearly install all required runtime files.
- Include uninstall path.
- Include third-party notices.
- Include minimal diagnostic logging and a support data collection command.
- Document unsupported hosts, OS versions, architectures, bit depths, contexts, and media types.
