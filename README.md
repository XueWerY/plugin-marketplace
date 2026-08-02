# 插件市场

地球 Online 生存日记内置社区插件市场，所有插件仓库均托管在 GitHub 上。

- [文件资源管理器](https://github.com/XueWerY/file-manager) — 文件浏览、数据导出与导入（局域网传输） #文件管理 #导出导入 #局域网传输
- [随机数生成器](https://github.com/XueWerY/random-number) — 在指定范围内生成随机数 #随机数 #实用工具
- [提醒查看器](https://github.com/XueWerY/reminder-viewer) — 查看所有已调度的提醒 #提醒 #实用工具

## 插件开发指南

插件是一个独立的 GitHub 仓库，放置到应用的 `src/plugins/` 目录下即可被自动加载。以下为完整的开发步骤。

### 一、目录结构

```
my-plugin/                  # 仓库根目录
├── plugin.json             # 插件元数据
├── index.ts                # 插件入口（导出 manifest + tools）
├── tools/                  # 小工具目录（可选）
│   └── my-tool/
│       └── index.vue       # 工具组件
├── .gitignore
├── README.md
└── LICENSE
```

### 二、plugin.json

插件元数据文件，定义插件基本信息及所含工具：

```json
{
  "id": "my-plugin",
  "name": "我的插件",
  "version": "1.0.0",
  "description": "插件的简要描述",
  "author": "你的名字",
  "tools": {
    "my-tool": {
      "name": "我的工具",
      "description": "工具的简要说明",
      "icon": "🔧"
    }
  }
}
```

| 字段 | 说明 |
|------|------|
| `id` | 唯一标识，使用小写连字符格式 |
| `name` | 插件显示名称 |
| `version` | 语义化版本号 |
| `description` | 简要描述 |
| `author` | 作者名称 |
| `tools` | 可选，工具映射表。key 为工具目录名，value 含 `name`（显示名）、`description`（说明）、`icon`（emoji 图标） |

### 三、index.ts

插件入口文件，固定写法如下：

```ts
import type { Component } from 'vue'
import manifest from './plugin.json'

const toolModules = import.meta.glob<{ default: Component }>('./tools/*/index.vue')

const toolsMeta = (manifest as any).tools || {}

const tools = Object.entries(toolModules).map(([path, loader]) => {
  const dirName = path.split('/')[2]
  const meta = toolsMeta[dirName] || {}
  return {
    id: `${manifest.id}/${dirName}`,
    pluginId: manifest.id,
    name: meta.name || dirName,
    description: meta.description || '',
    icon: meta.icon || '🔧',
    component: loader as () => Promise<Component>,
  }
})

export default { manifest, tools }
```

### 四、工具组件（tools/my-tool/index.vue）

一个标准的 Vue 3 单文件组件，在工具箱中点击卡片后会全屏打开。可使用 Element Plus 组件库和项目变量（`--chalk-*` 等）。无需手动注册，由插件系统自动加载。

```vue
<template>
  <div class="my-tool">
    <h2>我的工具</h2>
    <p>工具内容</p>
  </div>
</template>

<script setup lang="ts">
// 工具逻辑
</script>

<style scoped>
.my-tool {
  color: var(--chalk-white-90);
}
</style>
```

### 五、本地测试

1. 将插件目录复制到 `src/plugins/` 下
2. 启动应用：`npx vite`
3. 打开"工具箱"页面，在"小工具"区域可看到你的工具卡片
4. 点击卡片进入工具页面

### 六、发布到插件市场

1. 将插件仓库上传到 GitHub
2. Fork 本仓库（[XueWerY/plugin-marketplace](https://github.com/XueWerY/plugin-marketplace)）
3. 按以下格式将你的插件添加到 README.md 的插件列表中：
   ```
   - [插件名称](GitHub 仓库 URL) — 一句话描述 #标签1 #标签2
   ```
   | 部分 | 说明 |
   |------|------|
   | 插件名称 | 卡片上显示的名称 |
   | 仓库 URL | 插件 GitHub 地址，需含 `plugin.json` |
   | 描述 | 一句话说明功能 |
   | 标签 | 空格分隔的 `#` 标签，用于分类检索 |
4. 提交 Pull Request

