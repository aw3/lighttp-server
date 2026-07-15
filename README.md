# lighttp-server

> 轻量级静态文件服务器，**零运行时依赖**。支持 Markdown 实时预览、目录浏览、端口自动递增。

[![npm version](https://img.shields.io/npm/v/lighttp-server.svg)](https://www.npmjs.com/package/lighttp-server)
[![license](https://img.shields.io/npm/l/lighttp-server.svg)](https://github.com/aw3/lighttp-server/blob/main/LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D12.0.0-brightgreen.svg)](https://nodejs.org/)

---

## 特性

- **零依赖** — 仅使用 Node.js 内置模块，无需安装任何第三方包
- **静态文件托管** — 支持 HTML、CSS、JS、图片、字体等常见文件类型
- **Markdown 实时预览** — 访问 `.md` 文件自动渲染为带语法高亮的 HTML 页面，支持一键下载原始文件
- **目录浏览模式** — 可选的列表视图，展示文件大小、修改时间，支持排序和返回上级
- **端口自动递增** — 当指定端口被占用时，自动尝试下一个端口（最多重试 5 次）
- **安全防护** — 内置目录遍历防护和隐藏文件过滤（`.git`、`.env` 等不可访问）
- **中文界面** — CLI 输出和 HTML 模板均为中文

---

## 安装

### 全局安装（推荐）

```bash
npm install -g lighttp-server
```

### 本地安装

```bash
npm install lighttp-server
npx lighttp-server
```

### 不安装直接运行

```bash
npx lighttp-server
```

---

## 使用

### 基本用法

```bash
# 使用默认配置（端口 8080，当前目录）
lighttp-server

# 指定端口
lighttp-server -p 3000

# 指定根目录
lighttp-server -r ./dist

# 同时指定端口和目录
lighttp-server -p 9000 -r ./docs
```

### 目录浏览模式

```bash
# 启用目录列表（显示文件大小、修改时间、支持下载）
lighttp-server -l

# 浏览指定目录
lighttp-server -l -r ./docs
```

### CLI 选项

| 选项 | 说明 |
|------|------|
| `-p, --port <端口>` | 指定端口号（默认 8080，范围 1024–65535） |
| `-r, --root <目录>` | 指定根目录（默认当前目录） |
| `-l, --list` | 启用目录浏览模式（列出目录内容，支持文件下载） |
| `-h, --help` | 显示帮助信息 |

---

## 示例

### 1. 预览 Markdown 文档

```bash
lighttp-server -r ./docs -p 4000
```

浏览器访问 `http://localhost:4000/README.md`：

- 默认展示渲染后的 HTML 页面，带语法高亮
- 点击页面上的「下载 .md」按钮可获取原始 Markdown 文件

### 2. 作为项目本地开发服务器

```bash
lighttp-server -p 3000 -r ./build
```

等同于访问 `http://localhost:3000` 查看 `./build` 目录下的内容。

### 3. 启用目录浏览

```bash
lighttp-server -l -r ./downloads
```

浏览器将展示文件列表，包含文件大小、修改时间，点击即可下载。

---

## 安全特性

- **目录遍历防护** — 禁止访问根目录之外的文件（如 `../package.json` 返回 403）
- **隐藏文件过滤** — 路径中以 `.` 开头的文件和目录均返回 404，避免暴露敏感文件存在性
- **端口范围校验** — 仅允许 1024–65535 的端口号，防止误用特权端口

---

## 技术细节

| 项目 | 说明 |
|------|------|
| 运行时 | Node.js >= 12.0.0 |
| 核心模块 | `http`, `fs`, `path`, `url`（全部内置） |
| Markdown 渲染 | 浏览器端 `markdown-it` + `highlight.js`（内置在 vendor 目录） |
| 包体积 | 无第三方依赖，安装体积极小 |

---

## 许可证

[MIT](LICENSE)
