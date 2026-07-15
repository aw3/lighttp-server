# AGENTS.md — lighttp-server

> 本文档面向 AI 编码助手。读者应被视作对该项目一无所知。

---

## 项目概述

`lighttp-server` 是一个轻量级静态文件服务器，**零运行时依赖**。基于 Node.js 内置的 `http` 模块实现，支持：

- 静态文件托管（HTML、CSS、JS、图片、字体等）
- **Markdown 实时预览**（默认访问 `.md` 文件时渲染为 HTML，支持语法高亮）
- **目录浏览模式**（`-l` / `--list` 参数，列出目录内容并支持文件下载）
- **端口自动递增**（当指定端口被占用时，自动尝试下一个端口，最多重试 5 次）
- 中文界面与错误提示

项目语言：代码注释、CLI 输出、HTML 模板均为**中文**。

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 运行时 | Node.js >= 12.0.0 |
| 核心模块 | `http`, `fs`, `path`, `url`（全部内置，零第三方依赖） |
| Markdown 渲染 | 浏览器端 `markdown-it` + `highlight.js`（vendor 目录内置） |
| CLI 入口 | `#!/usr/bin/env node`，通过 `package.json` 的 `bin` 字段暴露 |

---

## 目录结构

```
lighttp-server/
├── index.js              # 唯一入口：CLI 参数解析 + HTTP 服务器 + 路由逻辑
├── package.json          # 包配置、bin 入口、引擎约束
├── templates/
│   ├── markdown.html     # Markdown 预览页面模板（含 toolbar + markdown-it 渲染）
│   └── directory.html    # 目录浏览列表页面模板
└── vendor/
    ├── github.min.css    # Markdown 预览的 GitHub 风格 CSS
    ├── highlight.min.js  # 代码语法高亮库
    ├── markdown-it.min.js      # Markdown 渲染引擎
    ├── markdown-it.min.js.map  # Source map（可选）
    └── marked.min.js     # 备用 Markdown 渲染库（当前未使用）
```

**关键设计**：`index.js` 是单文件应用，所有逻辑（参数解析、路由、安全校验、模板渲染、端口监听）集中于此。没有 `src/`、`lib/` 等分层目录。

---

## 启动与构建

### 本地开发启动

```bash
node index.js [选项] [目录]
```

或安装后作为全局 CLI 使用：

```bash
npm install -g .
lighttp-server -p 3000 -r ./dist
```

### CLI 选项

| 选项 | 说明 |
|------|------|
| `-p, --port <端口>` | 指定端口号（默认 8080，范围 1024–65535） |
| `-r, --root <目录>` | 指定根目录（默认当前目录） |
| `-l, --list` | 启用目录浏览模式（列出目录文件，支持下载） |
| `-h, --help` | 显示帮助信息 |

### 构建

本项目**无需构建步骤**。`npm start` 即 `node index.js`。发布时 `files` 字段仅包含 `index.js`、`vendor/`、`templates/`。

---

## 核心逻辑与路由

`index.js` 中的 `http.createServer` 处理所有请求：

1. **URL 解析** → 解码 pathname，提取 query 参数。
2. **Vendor 资源** → 以 `/vendor/` 开头的请求从**包自身目录**（`__dirname`）读取，而非用户根目录。
3. **安全检查**（两道防线）：
   - **目录遍历防护**：`path.resolve(filePath)` 必须位于 `resolvedRoot` 之下，否则返回 403。
   - **隐藏文件过滤**：路径中任何以 `.` 开头的段（如 `.git/`、`.env`）均返回 404，避免暴露敏感文件存在。
4. **文件读取** → 成功则按 MIME 类型返回；失败分情况处理：
   - `ENOENT` → 404
   - `EISDIR`（目录）→
     - **list 模式**：渲染目录文件列表 HTML（支持排序、返回上级、文件大小/时间展示）。
     - **非 list 模式**：尝试读取目录内的 `index.html`，失败则 404。
5. **Markdown 特殊处理**（`.md` 文件）：
   - 默认：通过 `markdown-it` + `highlight.js` 在浏览器端渲染为 HTML 预览页。
   - `?download=true`：返回原始 `.md` 文件，并触发浏览器下载。

### 端口监听策略

- 原始端口被占用时，自动尝试 `port + 1`，最多重试 5 次。
- 若均失败，打印被占用的端口列表并退出（code 1）。
- 成功启动后打印根目录绝对路径和访问地址。

---

## 代码风格与约定

- **单文件架构**：所有功能集中在 `index.js`，没有模块拆分。新增功能时通常直接在该文件内追加函数。
- **模板字符串替换**：HTML 模板使用 `{{placeholder}}` 风格占位符，通过 `String.prototype.replace` 批量替换。
- **同步读取模板**：启动时一次性同步读取 `templates/` 和 `vendor/` 文件到内存，运行时不再磁盘 IO。
- **中文输出**：所有 `console.log` / `console.error`、HTML 模板文案、注释均为中文。
- **无 linter / formatter 配置**：项目中没有 `.eslintrc`、`.prettierrc` 等文件。

---

## 测试策略

**当前项目没有自动化测试**（无 `test/` 目录、无测试框架依赖、无 `package.json` test script）。

验证方式以**手动测试**为主：

1. 启动服务器：`node index.js -p 3000 -r ./test-dir -l`
2. 浏览器访问 `http://localhost:3000`，检查：
   - 静态文件（HTML、CSS、图片）正常返回
   - `.md` 文件默认渲染为预览页，点击“下载 .md”可下载原始文件
   - 目录浏览模式下列表排序、返回上级、文件大小/时间显示正确
   - 访问 `http://localhost:3000/../package.json` 应返回 403（目录遍历防护）
   - 访问 `http://localhost:3000/.git/config` 应返回 404（隐藏文件过滤）
3. 端口占用测试：启动两个实例，第二个应自动递增端口并提示。

---

## 安全注意事项

- **目录遍历**：已做 `path.resolve` 前缀校验，禁止访问 `resolvedRoot` 之外的文件。
- **隐藏文件暴露**：任何路径段以 `.` 开头均返回 404（而非 403），避免泄露文件存在性信息。
- **端口号校验**：CLI 参数中的端口必须是数字，且限制在 1024–65535 范围，防止特权端口或无效值。
- **无身份验证**：这是一个纯静态文件服务器，没有登录、Cookie、Session 等机制。若用于生产环境，应前置反向代理（如 Nginx）并自行添加访问控制。

---

## 扩展建议（供 AI 助手参考）

- 如需新增 CLI 选项，在 `args` 解析循环中添加分支，并更新帮助文本。
- 如需新增 MIME 类型，在 `MIME_TYPES` 对象中追加映射。
- 如需修改 Markdown 预览样式，编辑 `templates/markdown.html` 中的 `<style>` 或替换 `vendor/github.min.css`。
- 如需修改目录列表样式，编辑 `templates/directory.html`。
- 如需拆分代码，当前单文件架构下建议保持简单，避免引入构建工具；若功能显著膨胀，可考虑将路由、模板、CLI 解析拆分为同级 `.js` 文件并通过 `require` 引入。
