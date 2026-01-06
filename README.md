# Markdown Preview Enhanced Plus

[![Version](https://img.shields.io/badge/version-0.8.20-blue.svg)](https://github.com/nanyeglm/markdown-preview-enhanced-plus)
[![License](https://img.shields.io/badge/license-NCSA-green.svg)](LICENSE.md)

基于 [Markdown Preview Enhanced](https://github.com/shd101wyy/vscode-markdown-preview-enhanced) 的增强版本，新增**自动预览管理**功能。

## ✨ 新增功能

相比原版 Markdown Preview Enhanced，本项目新增了以下核心功能：

### 1. 🔄 打开 Markdown 时自动侧边预览

打开任意 `.md` 文件时，自动在右侧打开预览窗口，无需手动触发。

### 2. 🚪 关闭 Markdown 文件时自动关闭预览

关闭 Markdown 源文件时，对应的预览窗口会自动关闭，保持编辑器整洁。

### 3. 🔀 切换到非 Markdown 文件时自动关闭预览

当切换到其他类型文件（如 `.py`、`.js`、`.json` 等）时，预览窗口自动关闭。

### 4. 🔒 预览编辑器组锁定

预览窗口所在的编辑器组会被锁定，防止其他标签页意外进入预览区域。新打开的文件始终在左侧主编辑区显示。

## 🎯 解决的痛点

| 原版问题 | 本版解决方案 |
|---------|-------------|
| 关闭 Markdown 后预览窗口残留 | 自动关闭预览 |
| 切换文件后预览内容过时 | 自动关闭或更新预览 |
| 每次都要手动打开预览 | 自动侧边预览 |
| 新文件可能出现在预览栏 | 锁定预览组 |

## ⚙️ 配置选项

在 VSCode 设置中搜索 `markdown-preview-enhanced`，或在 `settings.json` 中添加：

```json
{
  "markdown-preview-enhanced.automaticallyShowPreviewOfMarkdownBeingEdited": true,
  "markdown-preview-enhanced.closePreviewOnNonMarkdown": true,
  "markdown-preview-enhanced.closePreviewOnMarkdownClose": true
}
```

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `automaticallyShowPreviewOfMarkdownBeingEdited` | 打开 Markdown 时自动侧边预览 | `false` |
| `closePreviewOnNonMarkdown` | 切换到非 Markdown 文件时关闭预览 | `false` |
| `closePreviewOnMarkdownClose` | 关闭 Markdown 源文件时关闭预览 | `false` |

## 🔧 性能优化

为了提升用户体验，本项目还进行了以下性能优化：

- **预热机制**：插件激活时在后台预初始化 PreviewProvider，减少首次打开延迟
- **防抖机制**：快速切换文件时使用 150ms 防抖，避免竞争条件
- **版本控制**：确保异步操作只处理最新的编辑器切换事件

## 📦 安装方式

### 方式一：从 VSIX 安装

1. 下载 [Releases](https://github.com/nanyeglm/markdown-preview-enhanced-plus/releases) 中的 `.vsix` 文件
2. 在 VSCode 中按 `Ctrl+Shift+P`
3. 输入 `Extensions: Install from VSIX...`
4. 选择下载的 `.vsix` 文件
5. 重启 VSCode

### 方式二：从源码构建

详见下方 [从源码构建](#从源码构建) 章节。

## 🛠️ 从源码构建

### 环境要求

- Node.js 18+
- Yarn 1.22+
- Git

### 构建步骤

```bash
# 1. 克隆仓库
git clone https://github.com/nanyeglm/markdown-preview-enhanced-plus.git
cd markdown-preview-enhanced-plus

# 2. 安装依赖
yarn install

# 3. 构建项目
yarn build

# 4. 打包 VSIX
# 首先安装 vsce（如果尚未安装）
npm install -g @vscode/vsce

# 打包
vsce package --no-dependencies

# 5. 安装生成的 .vsix 文件
# 在 VSCode 中: Ctrl+Shift+P -> Extensions: Install from VSIX...
```

### 开发调试

```bash
# 在 VSCode 中打开项目
code .

# 按 F5 启动调试模式
# 这将打开一个新的 VSCode 窗口用于测试插件
```

## 📁 项目结构

```
├── src/
│   ├── extension-common.ts    # 核心扩展逻辑（包含自动预览功能）
│   ├── preview-provider.ts    # 预览提供者
│   ├── config.ts              # 配置管理
│   └── ...
├── docs/
│   └── OPTIMIZATION_SUMMARY.md  # 优化过程详细文档
├── package.json               # 插件配置和依赖
└── README.md                  # 本文件
```

## 🔗 相关链接

- **原版插件**：[Markdown Preview Enhanced](https://github.com/shd101wyy/vscode-markdown-preview-enhanced)
- **参考插件**：[Auto Markdown Preview Lock](https://github.com/nicepkg/auto-markdown-preview-lock)
- **优化文档**：[docs/OPTIMIZATION_SUMMARY.md](docs/OPTIMIZATION_SUMMARY.md)

## 📄 许可证

本项目基于 [University of Illinois/NCSA Open Source License](LICENSE.md) 开源。

原版 Markdown Preview Enhanced 由 [shd101wyy](https://github.com/shd101wyy) 开发。

## 🙏 致谢

- [shd101wyy](https://github.com/shd101wyy) - Markdown Preview Enhanced 原作者
- [nicepkg](https://github.com/nicepkg) - Auto Markdown Preview Lock 作者

---

**如果这个项目对你有帮助，欢迎 Star ⭐**
