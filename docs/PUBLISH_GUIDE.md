# 发布到 VSCode 插件商店 / Open VSX Registry 教程

本文档介绍如何将 Markdown Preview Enhanced Plus 发布到 VSCode Marketplace 或 Open VSX Registry。

## 一、发布到 VSCode Marketplace

### 1.1 前置准备

#### 1.1.1 创建 Azure DevOps 组织

1. 访问 [Azure DevOps](https://dev.azure.com/)
2. 使用 Microsoft 账号登录（可以用 GitHub 账号关联）
3. 创建一个组织（Organization），例如 `nanyeglm`

#### 1.1.2 创建 Personal Access Token (PAT)

1. 登录 Azure DevOps 后，点击右上角用户图标
2. 选择 **Personal access tokens**
3. 点击 **+ New Token**
4. 填写信息：
   - **Name**: `vscode-marketplace`
   - **Organization**: 选择你的组织
   - **Expiration**: 选择合适的过期时间
   - **Scopes**: 选择 **Custom defined**，然后：
     - 展开 **Marketplace**
     - 勾选 **Manage**（管理权限）
5. 点击 **Create**
6. **重要**：复制并保存生成的 Token，它只显示一次！

#### 1.1.3 创建 Publisher

1. 访问 [VSCode Marketplace Publisher Management](https://marketplace.visualstudio.com/manage)
2. 点击 **Create publisher**
3. 填写信息：
   - **Name**: `nanyeglm`（显示名称）
   - **ID**: `nanyeglm`（唯一标识符，用于插件 ID）
4. 点击 **Create**

### 1.2 修改 package.json

确保 `package.json` 中的 `publisher` 字段与你的 Publisher ID 一致：

```json
{
  "name": "markdown-preview-enhanced-plus",
  "displayName": "Markdown Preview Enhanced Plus",
  "publisher": "nanyeglm",
  "version": "0.8.20",
  ...
}
```

### 1.3 发布插件

#### 方式一：使用 vsce 命令行

```bash
# 安装 vsce（如果尚未安装）
npm install -g @vscode/vsce

# 登录（使用之前创建的 PAT）
vsce login nanyeglm
# 输入 Personal Access Token

# 发布
vsce publish

# 或者指定版本号发布
vsce publish 0.8.20

# 或者递增版本号并发布
vsce publish minor  # 0.8.20 -> 0.9.0
vsce publish patch  # 0.8.20 -> 0.8.21
```

#### 方式二：手动上传 VSIX

1. 打包 VSIX 文件：

   ```bash
   vsce package --no-dependencies
   ```

2. 访问 [Publisher Management](https://marketplace.visualstudio.com/manage)
3. 选择你的 Publisher
4. 点击 **+ New extension** → **Visual Studio Code**
5. 上传生成的 `.vsix` 文件

### 1.4 更新插件

```bash
# 更新版本号并发布
vsce publish patch  # 递增补丁版本
# 或
vsce publish minor  # 递增次版本
# 或
vsce publish major  # 递增主版本
```

---

## 二、发布到 Open VSX Registry

[Open VSX Registry](https://open-vsx.org) 是一个开源的 VSCode 插件市场，适用于 VSCodium、Gitpod、Eclipse Theia 等编辑器。

### 2.1 前置准备

#### 2.1.1 创建 Open VSX 账号

1. 访问 [Open VSX Registry](https://open-vsx.org/)
2. 点击右上角 **Log in**
3. 使用 GitHub 账号登录

#### 2.1.2 创建 Access Token

1. 登录后，点击右上角用户名
2. 选择 **Access Tokens**
3. 点击 **Generate new token**
4. 输入 Token 描述，例如 `publish-token`
5. 点击 **Generate token**
6. **重要**：复制并保存生成的 Token！

#### 2.1.3 创建 Namespace

1. 访问 [Create Namespace](https://open-vsx.org/user-settings/namespaces)
2. 输入 Namespace 名称：`nanyeglm`
3. 点击 **Create**

### 2.2 修改 package.json

确保 `publisher` 字段与 Namespace 一致：

```json
{
  "publisher": "nanyeglm",
  ...
}
```

### 2.3 发布插件

#### 方式一：使用 ovsx 命令行

```bash
# 安装 ovsx
npm install -g ovsx

# 发布（需要先打包 VSIX）
vsce package --no-dependencies
ovsx publish markdown-preview-enhanced-plus-0.8.20.vsix -p <your-token>

# 或者直接从源码发布
ovsx publish -p <your-token>
```

#### 方式二：手动上传

1. 打包 VSIX 文件
2. 访问 [Open VSX - Publish](https://open-vsx.org/user-settings/extensions)
3. 点击 **Publish Extension**
4. 上传 `.vsix` 文件

### 2.4 更新插件

```bash
# 更新 package.json 中的版本号
# 然后重新打包并发布
vsce package --no-dependencies
ovsx publish markdown-preview-enhanced-plus-x.x.x.vsix -p <your-token>
```

---

## 三、CI/CD 自动发布（可选）

可以配置 GitHub Actions 自动发布到两个平台。

### 3.1 设置 GitHub Secrets

在 GitHub 仓库中添加以下 Secrets：

1. 进入仓库 → **Settings** → **Secrets and variables** → **Actions**
2. 添加以下 Secrets：
   - `VSCE_PAT`: Azure DevOps Personal Access Token
   - `OVSX_PAT`: Open VSX Access Token

### 3.2 创建 GitHub Action

创建 `.github/workflows/publish.yml`：

```yaml
name: Publish Extension

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: yarn install

      - name: Build
        run: yarn build

      - name: Package
        run: npx vsce package --no-dependencies

      - name: Publish to VSCode Marketplace
        run: npx vsce publish -p ${{ secrets.VSCE_PAT }}

      - name: Publish to Open VSX
        run: npx ovsx publish -p ${{ secrets.OVSX_PAT }}
```

### 3.3 发布流程

1. 更新 `package.json` 中的版本号
2. 提交并推送更改
3. 在 GitHub 上创建 Release
4. GitHub Actions 自动构建并发布

---

## 四、常见问题

### Q1: vsce login 失败

确保 Personal Access Token 有 **Marketplace - Manage** 权限。

### Q2: 发布时提示 Publisher 不存在

确保 `package.json` 中的 `publisher` 字段与你在 Marketplace 创建的 Publisher ID 完全一致。

### Q3: 插件名称冲突

如果插件名称已被占用，需要修改 `package.json` 中的 `name` 字段。

### Q4: 如何撤销已发布的版本

在 Publisher Management 页面可以撤销（unpublish）已发布的版本，但建议发布新版本而不是撤销。

---

## 五、发布检查清单

发布前请确认以下事项：

- [ ] `package.json` 中的 `version` 已更新
- [ ] `package.json` 中的 `publisher` 正确
- [ ] `README.md` 内容完整
- [ ] `CHANGELOG.md` 已更新
- [ ] 本地测试通过
- [ ] 已执行 `yarn build` 构建
- [ ] `.vsix` 文件可以正常安装

---

## 六、相关链接

- [VSCode Extension Publishing](https://code.visualstudio.com/api/working-with-extensions/publishing-extension)
- [VSCode Marketplace Publisher Management](https://marketplace.visualstudio.com/manage)
- [Open VSX Registry](https://open-vsx.org/)
- [Open VSX Publishing Guide](https://github.com/eclipse/openvsx/wiki/Publishing-Extensions)
