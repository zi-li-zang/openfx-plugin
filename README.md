# OpenFX Plugin Skill

这是一个面向 Codex 的 OpenFX Image Effect 插件开发技能包，用于辅助设计、实现、调试、审查和文档化 `.ofx` 插件。它重点覆盖 OpenFX Image Effect API、C/C++ Support Library、OFX bundle 打包、宿主集成、temporal access、render 生命周期、pixel buffer、rowBytes、tiling、线程安全和商业分发检查。

这个 skill 不替代 OpenFX 官方文档。它的作用是把常见的 OpenFX 插件开发流程、审查顺序和工程经验整理成一个可复用的工作指南；当涉及“OpenFX 官方要求是什么”时，应优先核对 OpenFX 官网、官方 headers 或官方 examples。

## 适用场景

适合在以下任务中使用：

- 开发或审查 `.ofx` / `.ofx.bundle` 插件。
- 调试 DaVinci Resolve、Nuke、Natron、Baselight 等 OFX 宿主中的插件加载、渲染、参数或打包问题。
- 分析 `describe()`、`describeInContext()`、`createInstance()`、`render()`、`getFramesNeeded()`、`isIdentity()` 等 OFX 生命周期逻辑。
- 处理 OpenFX clips、parameters、contexts、temporal frame access、RoD/RoI、clip preferences。
- 排查 pixel buffer、bottom-left origin、rowBytes、render window、tiling、proxy、bit depth、component order 等图像问题。
- 审查外部 SDK / GPU runtime / ML 模型与 OFX 插件之间的封装边界。
- 准备商业分发时检查 bundle 结构、`install_name` / `rpath`、运行时 dylib、资源文件和第三方 license notices。

不适合：

- OpenFX Mesh Effect / OpenMfx。
- 非 OFX 的 DaVinci Workflow Integration 插件。
- 单纯的 DaVinci scripting / automation 工作流。
- 与 OpenFX Image Effect 无关的通用 C++ 或图像处理问题。

## 核心原则

这个 skill 使用分层判断方式：

1. **OpenFX 官方文档、官方 headers、官方 examples** 定义 OpenFX requirement。
2. **当前项目代码、构建文件、包内容、日志和测试结果** 定义项目真实行为。
3. **目标宿主文档、日志和实测行为** 定义 host-specific requirement。
4. **本 skill** 提供可复用流程、审查逻辑和工程建议。

因此：

- 简单、稳定、已沉淀在 skill 里的工作流逻辑，可以直接按 skill 执行。
- 涉及 OpenFX 规范结论时，必须核对官网或官方 headers/examples。
- 如果 skill 与官网冲突，以官网为准。
- 如果 skill 与当前项目证据冲突，先相信项目当前状态，并把 skill 视为需要更新的指导。
- 如果无法联网查看官网，应说明结论来自本地/offline references，不能直接当作最终规范结论。

## 目录结构

```text
openfx-plugin/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── official-contract.md
    ├── lifecycle-actions.md
    ├── parameters-ui.md
    ├── images-coordinates.md
    ├── suites-interacts.md
    ├── packaging-hosts.md
    └── support-library-patterns.md
```

### `SKILL.md`

Codex skill 的入口文件。它定义：

- skill 的触发范围。
- OpenFX 官方来源优先规则。
- references 的按需读取路由。
- OFX 插件开发/审查的通用 workflow。
- contract checklist、debugging order 和 production validation gate。

### `agents/openai.yaml`

用于 Codex UI 中显示 skill 名称、简介和默认提示。它不包含核心技术内容，但会影响 skill 在界面中的展示方式。

### `references/`

把详细知识拆成多个专题文件，避免每次使用时一次性加载全部内容。

| 文件 | 内容 |
| --- | --- |
| `official-contract.md` | OpenFX C ABI、导出符号、`OfxPlugin`、contexts、temporal coordinates、官方链接 |
| `lifecycle-actions.md` | `describe`、`describeInContext`、`render`、`isIdentity`、RoD/RoI、`getFramesNeeded`、status codes |
| `parameters-ui.md` | 参数类型、动画参数、choice / StrChoice、pages、groups、undo、saved project 兼容性 |
| `images-coordinates.md` | 坐标系统、image bounds、rowBytes、bottom-left origin、render window、tiling、fields、proxy |
| `suites-interacts.md` | OFX suites、property sets、image memory、message/progress/timeline、custom interact |
| `packaging-hosts.md` | bundle 结构、macOS 打包、runtime dylib、`install_name` / `rpath`、license notices、clean-machine 测试 |
| `support-library-patterns.md` | OpenFX C++ Support Library 与 raw C action 的对应关系、常见 override pattern |

## 使用方式

把仓库放在 Codex skills 目录下：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/zi-li-zang/openfx-plugin.git ~/.codex/skills/openfx-plugin
```

在 Codex 中，当任务涉及 OpenFX 插件开发、调试、审查或文档时，这个 skill 会根据 `SKILL.md` 的描述触发。

也可以在提问时明确说明：

```text
使用 openfx-plugin skill，帮我审查这个 OFX 插件的 render / getFramesNeeded 逻辑。
```

## 典型提问

```text
这个 OFX 插件为什么 DaVinci 能加载但 render 不执行？
```

```text
帮我审查 getFramesNeeded() 和 render() 的 temporal access 是否一致。
```

```text
这个插件声明了支持 tiles，但算法需要整帧输入，这样安全吗？
```

```text
macOS OFX bundle 里的 dylib 应该怎么放，rpath 怎么检查？
```

```text
DaVinci 里图像上下颠倒，是 OpenFX 坐标系还是我的 pixel conversion 有问题？
```

## 官方来源

处理规范问题时，优先查看：

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

## 维护建议

更新这个 skill 时，建议遵守以下规则：

- 不要把宿主经验写成 OpenFX 官方要求。
- 不要把项目策略写成通用规范。
- 新增硬性判断前，先核对 OpenFX 官网、官方 headers 或官方 examples。
- 对 DaVinci、Nuke、Natron 等宿主差异，要明确标注为 host-specific。
- reference 文件可以沉淀经验，但 `SKILL.md` 应保持简洁，主要负责触发、路由和执行规则。
- 如果发现官网和 skill 现有内容冲突，应更新 skill 或降低表述强度。

## 许可证

本仓库目前只包含 Codex skill 文档和参考资料。具体许可证请根据仓库实际发布策略补充。
