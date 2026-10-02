# Precise-pixels

# PixelLock Studio

> AI 局部重绘工作台：像素级锁定，图层化编辑，多后端可插拔。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-early%20development-orange)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

PixelLock Studio 是一个开源的 AI 图像局部编辑工具。它的目标不是“生成一张新图”，而是让你像在 Photoshop 里一样，**精确控制 AI 只修改你指定的区域，其他像素保持不变**。

<!-- TODO: 替换为演示 GIF -->
![demo](docs/assets/demo.gif)

---

## 为什么做这个项目

现有 AI 图像编辑工具普遍存在四个问题：

- **牵一发而动全身**：只想改一个细节，结果背景、光影、构图全变了。
- **只能重来，不能回退局部**：一次不满意，只能整张重绘，之前的满意修改也丢了。
- **绑定单一模型**：用了某家 API，就无法切换到本地 ComfyUI 或其他模型。
- **与专业工作流割裂**：在 PS / Krita 里选好区，还要导出、上传、等待、下载、再贴回。

PixelLock Studio 试图把“精确控制”还给用户。它不是又一个生成器，而是 **AI 编辑的控制层**。

---

## 核心特性

### 1. 像素级锁定与区域保护

三层蒙版系统，让你精确决定每个像素的命运：

| 蒙版层 | 行为 | 用途 |
| :--- | :--- | :--- |
| **保护层** | 绝对不动，像素级锁定 | 人脸、Logo、文字、关键结构 |
| **柔化层** | Alpha 混合，边缘过渡 | 接缝、阴影、纹理融合 |
| **生成层** | 完全重绘 | 需要 AI 重新生成的内容 |

生成结果采用**严格贴回**策略：只替换生成层内的像素，保护层和柔化层外的区域与原因完全一致。

### 2. 迭代式局部编辑工作流

每次局部修改都作为一个独立图层保存：

- 可随时切换任意图层的可见性
- 可回退到任意历史节点，不影响其他局部编辑
- 编辑历史本地持久化，关闭浏览器不丢失
- 像 PS 图层一样管理 AI 编辑，而不是“生成→不满意→重来”

### 3. 多后端可插拔 + 本地优先

统一的 `InpaintBackend` 接口，后端可自由切换：

| 后端 | 类型 | 状态 |
| :--- | :--- | :--- |
| ComfyUI | 本地 | 规划中 |
| FLUX.1 Fill | API | 规划中 |
| 阿里云万相 | API | 规划中 |
| 即梦 AI | API | 规划中 |
| OpenAI / Google | API | 规划中 |

本地优先意味着：**你的图片可以不出本机**。对于电商、设计公司等对数据隐私敏感的场景，这是刚需。

### 4. 与传统编辑器桥接

规划中的桥接方案：

- **Photoshop UXP 插件**：PS 中选区 → 调用 PixelLock Studio → 结果自动回写为图层
- **Krita 插件**：Python 插件体系，接入成本更低
- **CLI / API**：支持脚本批量处理，集成到现有工具链

目标：在专业编辑器里完成精确选区，在 PixelLock Studio 里完成 AI 重绘，结果无缝回到原工作流。

---

## 项目状态

> **当前处于早期设计阶段。** 核心交互和架构正在搭建，欢迎参与讨论和贡献。

| 模块 | 状态 |
| :--- | :--- |
| 项目愿景与架构设计 | ✅ 完成 |
| Web 画布与蒙版系统 | 🚧 开发中 |
| 图层化编辑历史 | 📋 规划中 |
| ComfyUI 后端适配器 | 📋 规划中 |
| FLUX.1 Fill 后端适配器 | 📋 规划中 |
| Photoshop / Krita 桥接 | 📋 规划中 |

---

## 快速开始

> 以下命令为预期使用方式，部分功能尚未实现。MVP 完成后会更新本节。

### Docker Compose（推荐）

```bash
git clone https://github.com/yourname/pixellock-studio.git
cd pixellock-studio
cp .env.example .env
# 编辑 .env，填入你的 API Key 或 ComfyUI 地址
docker compose up
```

访问 `http://localhost:3000`。

### 本地开发

**前端**

```bash
cd apps/web
npm install
npm run dev
```

**后端**

```bash
cd apps/server
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

### 配置 ComfyUI 后端

在 `.env` 中设置：

```env
COMFYUI_BASE_URL=http://127.0.0.1:8188
DEFAULT_BACKEND=comfyui
```

将 `comfyui_workflows/` 中的工作流导入 ComfyUI，即可开始使用本地推理。

---

## 使用流程

---

## 架构概览

```text
pixellock-studio/
├── apps/
│   ├── web/              # React + Vite + Konva 画布编辑器
│   ├── server/           # FastAPI 后端代理 + 后端适配器
│   └── plugins/
│       ├── ps-uxp/       # Photoshop UXP 插件（规划中）
│       └── krita/        # Krita Python 插件（规划中）
├── packages/
│   ├── core/             # 蒙版系统、历史栈、后端接口定义
│   └── backends/         # ComfyUI / FLUX Fill / Wanx 适配器
├── comfyui_workflows/    # 预置工作流 JSON
├── docker/
├── docs/
└── examples/
```

**设计原则**

- **本地优先**：默认使用本地 ComfyUI，API 后端为可选项
- **后端无关**：核心逻辑不绑定任何模型供应商
- **像素级控制**：保护层像素绝对不变
- **工作流友好**：与 PS / Krita / CLI 无缝协作

---

## 后端适配器接口

---

## 路线图


## 参与贡献

欢迎任何形式的贡献：提 Issue、讨论设计、提交 PR、完善文档。

1. Fork 本仓库
2. 创建分支：`git checkout -b feature/your-feature`
3. 提交改动：`git commit -m 'feat: add something'`
4. 推送分支：`git push origin feature/your-feature`
5. 发起 Pull Request

请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 和 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)。

---

## 致谢

本项目的设计受到以下开源项目和研究的启发：

- [Scumble](https://github.com/scumble) — 桌面端 AI 局部重绘编辑器
- [RefineAnything](https://arxiv.org/) — 区域级精确图像修复
- [Auto-Photoshop-StableDiffusion-Plugin](https://github.com/AbdullahAlfaraj/Auto-Photoshop-StableDiffusion-Plugin) — PS 桥接方案
- [ComfyUI](https://github.com/comfyanonymous/ComfyUI) — 本地推理工作流
- [Prompt Canvas](https://github.com/) — 迭代式生成工作流

---

## 许可证

[MIT](LICENSE)

---

## 免责声明

本项目仅用于合法的图像编辑用途。请遵守所使用模型和 API 的服务条款。
