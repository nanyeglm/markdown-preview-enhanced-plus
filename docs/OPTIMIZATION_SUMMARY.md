# Markdown Preview Enhanced 自动预览功能优化总结

## 一、项目背景

### 1.1 原始问题

在使用 VSCode 的 Markdown Preview Enhanced (MPE) 插件时，存在以下用户体验问题：

1. **预览不会自动关闭**：关闭 Markdown 源文件后，右侧预览窗口仍然存在，影响后续编辑
2. **切换文件预览残留**：切换到非 Markdown 文件时，预览窗口依然显示之前的内容
3. **需要手动打开预览**：每次打开 Markdown 文件都需要手动触发侧边预览
4. **预览组被其他标签占用**：在预览区操作后，新打开的文件会出现在预览栏中

### 1.2 参考插件

[Auto Markdown Preview Lock](https://marketplace.visualstudio.com/items?itemName=Knworks.auto-markdown-preview-lock) 插件实现了上述功能，但其预览主题不符合需求（需要 MPE 的白底黑字主题）。

### 1.3 解决方案

在 MPE 源码基础上进行改造，引入 Auto Markdown Preview Lock 的核心功能，同时保持 MPE 原有的渲染样式和功能。

---

## 二、核心功能实现

### 2.1 功能一：关闭 Markdown 文件时自动关闭预览

#### 实现思路

监听 VSCode 的 `onDidCloseTextDocument` 事件，当检测到关闭的文档是 Markdown 文件时，自动关闭对应的预览窗口。

#### 关键代码

```typescript
// src/extension-common.ts
context.subscriptions.push(
  vscode.workspace.onDidCloseTextDocument(async (document) => {
    const closePreviewOnMarkdownClose = getMPEConfig<boolean>(
      'closePreviewOnMarkdownClose',
    );
    if (closePreviewOnMarkdownClose && isMarkdownFile(document)) {
      const previewProvider = await getPreviewContentProvider(document.uri);
      await unlockPreviewGroup();
      previewProvider.closeCurrentPreview(document.uri);
    }
  }),
);
```

#### 配置项

```json
"markdown-preview-enhanced.closePreviewOnMarkdownClose": {
  "description": "Automatically close markdown preview when the source markdown file is closed.",
  "default": false,
  "type": "boolean"
}
```

---

### 2.2 功能二：切换到非 Markdown 文件时自动关闭预览

#### 实现思路

监听 VSCode 的 `onDidChangeActiveTextEditor` 事件，当检测到当前活动编辑器不是 Markdown 文件时，自动关闭预览窗口。

#### 关键代码

```typescript
// src/extension-common.ts (在 onDidChangeActiveTextEditor 回调中)
} else {
  // Non-markdown file: close preview if configured
  const closePreviewOnNonMarkdown = getMPEConfig<boolean>(
    'closePreviewOnNonMarkdown',
  );
  if (closePreviewOnNonMarkdown) {
    const previewProvider = await getPreviewContentProvider(
      editor.document.uri,
    );
    if (previewProvider.hasAnyPreviewOpen()) {
      await unlockPreviewGroup();
      previewProvider.closeCurrentPreview();
    }
  }
}
```

#### 配置项

```json
"markdown-preview-enhanced.closePreviewOnNonMarkdown": {
  "description": "Automatically close markdown preview when switching to a non-markdown file.",
  "default": false,
  "type": "boolean"
}
```

---

### 2.3 功能三：打开 Markdown 时自动侧边预览

#### 实现思路

利用 MPE 已有的 `automaticallyShowPreviewOfMarkdownBeingEdited` 配置项，在 `onDidChangeActiveTextEditor` 事件中检测到 Markdown 文件时，自动调用 `openPreviewToTheSide` 方法。

#### 关键代码

```typescript
// src/extension-common.ts
if (isMarkdownFile(editor.document)) {
  // ...
  if (automaticallyShowPreviewOfMarkdownBeingEdited) {
    openPreviewToTheSide(sourceUri);
  }
}
```

#### 配置项

```json
"markdown-preview-enhanced.automaticallyShowPreviewOfMarkdownBeingEdited": {
  "description": "Automatically show preview of markdown being edited.",
  "default": false,
  "type": "boolean"
}
```

---

### 2.4 功能四：锁定预览编辑器组

#### 实现思路

参考 Auto Markdown Preview Lock 的实现，使用 VSCode 的 `workbench.action.lockEditorGroup` 命令锁定预览所在的编辑器组，防止其他标签页进入。同时，当检测到非 Markdown 文件在预览组中打开时，使用 `workbench.action.moveEditorToFirstGroup` 将其移动到主编辑区。

#### 关键代码

```typescript
// src/extension-common.ts

/**
 * Lock the preview editor group (column 2) to prevent other tabs from opening in it.
 */
async function lockPreviewGroup(fallbackEditor: vscode.TextEditor) {
  try {
    await vscode.commands.executeCommand('workbench.action.focusSecondEditorGroup');
    await vscode.commands.executeCommand('workbench.action.lockEditorGroup');
    await vscode.window.showTextDocument(fallbackEditor.document, {
      viewColumn: fallbackEditor.viewColumn,
      preserveFocus: false,
      preview: false,
    });
  } catch (error) {
    console.warn('[MPE] Failed to lock preview group:', error);
  }
}

/**
 * Unlock the preview editor group.
 */
async function unlockPreviewGroup() {
  try {
    await vscode.commands.executeCommand('workbench.action.focusSecondEditorGroup');
    await vscode.commands.executeCommand('workbench.action.unlockEditorGroup');
    await vscode.commands.executeCommand('workbench.action.focusFirstEditorGroup');
  } catch (error) {
    console.warn('[MPE] Failed to unlock preview group:', error);
  }
}

/**
 * Move the current editor to the first (primary) editor group.
 */
async function moveEditorToPrimaryGroup() {
  try {
    await vscode.commands.executeCommand('workbench.action.moveEditorToFirstGroup');
  } catch (error) {
    console.warn('[MPE] Failed to move editor to first group:', error);
  }
}
```

#### 调用时机

- **锁定**：在 `openPreviewToTheSide` 方法中，打开预览后立即锁定预览组
- **解锁**：在关闭预览前解锁，确保编辑器组可以正常关闭
- **移动**：当非 Markdown 文件在预览组中打开时，移动到主编辑区

---

## 三、性能优化

### 3.1 问题现象

在实际使用中发现两个问题：

1. **启动后响应慢**：刚打开项目后，打开 Markdown 文件不能立刻自动渲染
2. **快速切换失效**：在多个文件间快速切换时，自动预览功能偶尔失效

### 3.2 问题分析

| 问题 | 根本原因 |
|------|----------|
| 启动后响应慢 | `PreviewProvider` 首次调用需要初始化 `Notebook`（耗时操作） |
| 快速切换失效 | 无防抖机制，多个事件并发执行产生竞争条件 |

### 3.3 解决方案

#### 3.3.1 预热机制

在插件激活时，后台预初始化所有工作区的 `PreviewProvider`：

```typescript
/**
 * Preheat PreviewProvider for all workspace folders to reduce first-open delay.
 */
async function preheatPreviewProviders(context: vscode.ExtensionContext) {
  const workspaceFolders = vscode.workspace.workspaceFolders ?? [];
  for (const folder of workspaceFolders) {
    try {
      PreviewProvider.getPreviewContentProvider(folder.uri, context).catch(
        (error) => {
          console.warn('[MPE] Failed to preheat PreviewProvider:', error);
        },
      );
    } catch (error) {
      console.warn('[MPE] Error preheating PreviewProvider:', error);
    }
  }
}
```

#### 3.3.2 防抖机制

添加 150ms 的防抖延迟，避免快速切换时的无效处理：

```typescript
let autoPreviewDebounceTimer: NodeJS.Timeout | null = null;
let autoPreviewVersion = 0;
const AUTO_PREVIEW_DEBOUNCE_MS = 150;

// 在 onDidChangeActiveTextEditor 中
if (autoPreviewDebounceTimer) {
  clearTimeout(autoPreviewDebounceTimer);
  autoPreviewDebounceTimer = null;
}
autoPreviewVersion++;
const currentVersion = autoPreviewVersion;

autoPreviewDebounceTimer = setTimeout(async () => {
  if (currentVersion !== autoPreviewVersion) {
    return; // 版本不匹配，取消操作
  }
  // ... 实际处理逻辑
}, AUTO_PREVIEW_DEBOUNCE_MS);
```

#### 3.3.3 版本控制

使用版本号确保只处理最新的事件，异步操作完成后检查版本是否仍有效：

```typescript
const previewProvider = await getPreviewContentProvider(sourceUri);

// Check version again after async operation
if (currentVersion !== autoPreviewVersion) {
  return; // 过期的操作，直接返回
}
```

---

## 四、文件修改清单

| 文件 | 修改内容 |
|------|----------|
| `src/config.ts` | 添加 `closePreviewOnNonMarkdown` 和 `closePreviewOnMarkdownClose` 类型定义 |
| `package.json` | 添加两个新配置项的 JSON Schema 定义 |
| `src/preview-provider.ts` | 添加 `closeCurrentPreview` 和 `hasAnyPreviewOpen` 方法 |
| `src/extension-common.ts` | 添加事件监听、锁定/解锁函数、防抖机制、预热机制 |

---

## 五、配置推荐

在 VSCode 的 `settings.json` 中添加：

```json
{
  "markdown-preview-enhanced.automaticallyShowPreviewOfMarkdownBeingEdited": true,
  "markdown-preview-enhanced.closePreviewOnNonMarkdown": true,
  "markdown-preview-enhanced.closePreviewOnMarkdownClose": true
}
```

---

## 六、总结

本次优化成功将 Auto Markdown Preview Lock 的核心功能集成到 Markdown Preview Enhanced 中，实现了：

1. ✅ 关闭 Markdown 文件时自动关闭预览
2. ✅ 切换到非 Markdown 文件时自动关闭预览
3. ✅ 打开 Markdown 时自动侧边预览
4. ✅ 锁定预览编辑器组，防止其他标签进入
5. ✅ 通过预热和防抖机制优化性能

同时保持了 MPE 原有的白底黑字渲染主题和丰富的导出功能。
