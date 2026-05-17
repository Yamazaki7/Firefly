# Firefly Pio 看板娘配置维护文档

本文档根据 Firefly 官方文档和当前仓库实现整理，供后续使用 Codex 修改、维护 `Live2D` / `Spine` 看板娘配置时参考。

- 官方文档来源: https://docs-firefly.cuteleaf.cn/zh/guide/pio.html
- 官方文档源文件: https://github.com/CuteLeaf/Firefly-docs/edit/main/zh/guide/pio.md
- 本地配置入口: `src/config/pioConfig.ts`
- 类型定义入口: `src/types/config.ts`
- 组件实现:
  - `src/components/features/Live2DWidget.astro`
  - `src/components/features/SpineModel.astro`
- 本地资源目录: `public/pio/`
- 本文档整理日期: 2026-05-17

## 总览

Firefly 支持在页面角落显示看板娘模型，当前实现支持两类模型:

- `Live2D`: 使用 `live2dModelConfig`
- `Spine`: 使用 `spineModelConfig`

建议二选一启用，避免两个模型同时占据页面角落、重复加载运行时或产生交互冲突。

当前仓库默认状态:

```ts
// src/config/pioConfig.ts
export const spineModelConfig = {
  enable: false,
};

export const live2dModelConfig = {
  enable: true,
};
```

## 本地可用模型

当前仓库 `public/pio/` 下已经包含这些模型资源:

| 类型 | 配置路径 | 说明 |
| --- | --- | --- |
| Live2D | `/pio/models/live2d/snow_miku/model.json` | 当前默认启用 |
| Live2D | `/pio/models/live2d/illyasviel/illyasviel.model.json` | 可切换备用模型 |
| Spine | `/pio/models/spine/firefly/1310.json` | 需要同目录 `1310.atlas` 和贴图资源 |

`public` 目录下的文件在站点中以根路径访问，所以配置中应写 `/pio/...`，不要写 `public/pio/...`。

## Spine 配置

`spineModelConfig` 类型为 `SpineModelConfig`，用于控制 Spine 看板娘。

### 基础字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `enable` | `boolean` | `false` | 是否启用 Spine 看板娘 |
| `model.path` | `string` | `/pio/models/spine/firefly/1310.json` | Spine 骨骼 JSON 文件路径 |
| `model.scale` | `number` | `1.0` | 模型缩放比例 |
| `model.x` | `number` | `0` | X 轴偏移 |
| `model.y` | `number` | `0` | Y 轴偏移 |
| `zIndex` | `number` | `1000` | CSS 层级 |
| `opacity` | `number` | `1.0` | 透明度，范围通常为 `0` 到 `1` |

### 位置字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `position.corner` | `"bottom-left" \| "bottom-right" \| "top-left" \| "top-right"` | `"bottom-left"` | 模型所在角落 |
| `position.offsetX` | `number` | `0` | 水平边距，单位 px |
| `position.offsetY` | `number` | `0` | 垂直边距，单位 px |

右下角可能遮挡返回顶部按钮，优先使用 `"bottom-left"`。

### 尺寸字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `size.width` | `number` | `135` | 容器宽度，单位 px |
| `size.height` | `number` | `165` | 容器高度，单位 px |

尺寸控制的是外层容器。模型实际视觉大小还可能受 `model.scale`、Spine 数据和画布渲染影响。

### 交互字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `interactive.enabled` | `boolean` | `true` | 是否启用点击交互 |
| `interactive.clickAnimations` | `string[]` | `["emoji_0", "emoji_1", ...]` | 点击时随机播放的动画名列表 |
| `interactive.clickMessages` | `string[]` | 自定义文案数组 | 点击时随机显示的消息 |
| `interactive.messageDisplayTime` | `number` | `3000` | 消息显示时长，单位 ms |
| `interactive.idleAnimations` | `string[]` | `["idle", ...]` | 待机动画候选列表 |
| `interactive.idleInterval` | `number` | `8000` | 待机动画切换间隔，单位 ms |

本地 `SpineModel.astro` 会读取模型中的动画名并过滤不存在的动画。若配置了不存在的动画，浏览器控制台会输出警告，模型仍会尝试回退到可用的待机动画。

### 响应式字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `responsive.hideOnMobile` | `boolean` | `true` | 是否在移动端隐藏 |
| `responsive.mobileBreakpoint` | `number` | `768` | 移动端断点，单位 px |

当 `hideOnMobile` 为 `true` 且视口宽度小于等于断点时，组件会跳过 Spine 初始化以减少移动端资源开销。

### Spine 示例

```ts
export const spineModelConfig: SpineModelConfig = {
  enable: true,
  model: {
    path: "/pio/models/spine/firefly/1310.json",
    scale: 1.0,
    x: 0,
    y: 0,
  },
  position: {
    corner: "bottom-left",
    offsetX: 0,
    offsetY: 0,
  },
  size: {
    width: 135,
    height: 165,
  },
  interactive: {
    enabled: true,
    clickAnimations: ["emoji_0", "emoji_1", "emoji_2"],
    clickMessages: ["你好呀！", "今天也要加油哦！"],
    messageDisplayTime: 3000,
    idleAnimations: ["idle", "emoji_0"],
    idleInterval: 8000,
  },
  responsive: {
    hideOnMobile: true,
    mobileBreakpoint: 768,
  },
  zIndex: 1000,
  opacity: 1.0,
};
```

## Live2D 配置

`live2dModelConfig` 类型为 `Live2DModelConfig`，用于控制 Live2D 看板娘。

### 基础字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `enable` | `boolean` | `true` | 是否启用 Live2D 看板娘 |
| `model.path` | `string` | `/pio/models/live2d/snow_miku/model.json` | Live2D 模型 JSON 路径 |

本地实现会从模型 JSON 中读取 `motions` 和 `expressions`，通常不需要在配置里手动列出动作或表情。

### 位置字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `position.corner` | `"bottom-left" \| "bottom-right" \| "top-left" \| "top-right"` | `"bottom-left"` | 模型所在角落 |
| `position.offsetX` | `number` | `0` | 水平边距，单位 px |
| `position.offsetY` | `number` | `0` | 垂直边距，单位 px |

组件缺省值是 `bottom-right`、`20`、`20`，但当前配置文件显式设置为 `bottom-left`、`0`、`0`。

### 尺寸字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `size.width` | `number` | `135` | 容器宽度，单位 px |
| `size.height` | `number` | `165` | 容器高度，单位 px |

组件缺省尺寸是 `280 x 250`，当前配置文件显式设置为 `135 x 165`。

### 交互字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `interactive.enabled` | `boolean` | `true` | 是否启用点击交互 |
| `interactive.clickMessages` | `string[]` | 自定义文案数组 | 点击时随机显示的消息 |
| `interactive.messageDisplayTime` | `number` | `3000` | 消息显示时长，单位 ms |

当前 `Live2DWidget.astro` 会按点击区域尝试选择动作组:

- 点击头部时优先使用 `flick_head`
- 点击身体时优先使用 `tap_body`
- 如果目标动作组不存在，会回退到模型 JSON 中可用的动作组

### 响应式字段

| 字段 | 类型 | 当前/常用默认值 | 说明 |
| --- | --- | --- | --- |
| `responsive.hideOnMobile` | `boolean` | `true` | 是否在移动端隐藏 |
| `responsive.mobileBreakpoint` | `number` | `768` | 移动端断点，单位 px |

当移动端隐藏开启时，组件会跳过 Live2D SDK 初始化。

### Live2D 示例

```ts
export const live2dModelConfig: Live2DModelConfig = {
  enable: true,
  model: {
    path: "/pio/models/live2d/snow_miku/model.json",
  },
  position: {
    corner: "bottom-left",
    offsetX: 0,
    offsetY: 0,
  },
  size: {
    width: 135,
    height: 165,
  },
  interactive: {
    enabled: true,
    clickMessages: [
      "你好！我是Miku~",
      "有什么需要帮助的吗？",
      "记得按时休息哦！",
    ],
    messageDisplayTime: 3000,
  },
  responsive: {
    hideOnMobile: true,
    mobileBreakpoint: 768,
  },
};
```

## 切换模型的维护步骤

### 启用 Live2D，关闭 Spine

1. 修改 `src/config/pioConfig.ts`。
2. 设置 `live2dModelConfig.enable = true`。
3. 设置 `spineModelConfig.enable = false`。
4. 将 `live2dModelConfig.model.path` 指向一个存在的 `model.json` 或 `.model.json`。

可用路径示例:

```ts
path: "/pio/models/live2d/snow_miku/model.json"
path: "/pio/models/live2d/illyasviel/illyasviel.model.json"
```

### 启用 Spine，关闭 Live2D

1. 修改 `src/config/pioConfig.ts`。
2. 设置 `spineModelConfig.enable = true`。
3. 设置 `live2dModelConfig.enable = false`。
4. 将 `spineModelConfig.model.path` 指向 Spine JSON 文件。
5. 确保同目录存在同名 `.atlas` 文件和贴图资源。

可用路径示例:

```ts
path: "/pio/models/spine/firefly/1310.json"
```

## 资源放置规范

### Live2D

建议放到:

```txt
public/pio/models/live2d/<model-name>/
```

常见文件:

```txt
model.json
model.moc
textures/
motions/
```

或旧版 Live2D:

```txt
<model-name>.model.json
<model-name>.moc
textures/
motions/
```

### Spine

建议放到:

```txt
public/pio/models/spine/<model-name>/
```

常见文件:

```txt
<model-name>.json
<model-name>.atlas
<model-name>.png
images/
audio/
```

本地 `SpineModel.astro` 会根据 `model.path` 自动把 `.json` 替换为 `.atlas` 作为图集路径，所以同名 `.atlas` 很重要。

## 排错清单

### 模型不显示

- 检查对应配置的 `enable` 是否为 `true`。
- 确认 Live2D 和 Spine 没有同时启用。
- 确认 `model.path` 以 `/pio/` 开头，并且文件实际存在于 `public/pio/`。
- 打开浏览器控制台查看资源加载错误。
- 移动端测试时确认 `responsive.hideOnMobile` 和 `mobileBreakpoint` 是否导致组件隐藏。

### Spine 动画不播放

- 检查 `clickAnimations` 和 `idleAnimations` 是否与模型 JSON 中的动画名一致。
- 查看控制台是否有 `Spine click animations not found in model` 一类警告。
- 至少保留一个可用待机动画，例如 `idle`。

### Live2D 点击没有明显动作

- 检查模型 JSON 中是否存在 `motions`。
- 检查是否存在 `tap_body`、`flick_head` 等动作组。
- 某些 Live2D SDK 封装不支持直接切换动作，本地组件目前主要提供点击反馈、消息展示和动作组记录。

### 位置或遮挡问题

- 优先使用 `position.corner: "bottom-left"`。
- 调整 `offsetX`、`offsetY` 避免贴边或遮挡按钮。
- 调整 `size.width`、`size.height` 改变外层容器。
- Spine 可进一步尝试 `model.scale`、`model.x`、`model.y`。

## 维护建议

- 修改配置优先改 `src/config/pioConfig.ts`，不要直接改组件逻辑。
- 增加新模型时先确认授权和文件完整性。
- Spine 模型更换后，先从模型 JSON 中确认动画名，再更新 `clickAnimations` / `idleAnimations`。
- Live2D 模型更换后，先确认模型 JSON 能被浏览器直接访问。
- 改动后运行项目并在桌面端、移动端宽度各检查一次。
