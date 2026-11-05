---
tags:
  - 软件
  - 开发工具
  - AI
created: 2026-11-06
---

# 开发与 AI

> [!info] 导航
> 返回 [[软件清单]]

## NVM for Windows

**定位**：Windows 上的 Node.js 版本管理器。

**能干什么**

- 一台机器装多个 Node 版本，按项目切换
- 老项目锁 Node 16、新项目用 Node 20，互不干扰
- `nvm use` 即时切换，全局包按版本隔离
- 一条命令安装 / 卸载任意 Node 版本

**下载**

- GitHub Releases：<https://github.com/coreybutler/nvm-windows/releases>
- 选 `nvm-setup.exe`（安装版）或 `nvm-noinstall.zip`（免安装）
- winget：`winget install --id CoreyButler.NVMforWindows -e`

**常用命令**

```powershell
nvm list available      # 看可安装的版本
nvm install 20.11.0     # 装指定版本
nvm install lts         # 装最新 LTS
nvm list                # 看本地已装版本
nvm use 20.11.0         # 切换（需要管理员权限）
nvm current             # 当前版本
```

> [!warning] 装之前先卸掉原来的 Node
> 如果系统里已经装了官方 Node，nvm 会接管 `node` 命令但可能冲突。先卸载，再装 nvm。

> [!tip] 国内下载慢
> 在 nvm 安装目录的 `settings.txt` 里加一行 `node_mirror: https://npmmirror.com/mirrors/node/`，再 `nvm install` 会快很多。

## CC Switch

**定位**：管理 AI 编程工具（Claude Code、Codex、OpenCode 等）供应商配置的桌面助手。

**能干什么**

- 集中管理多个 API 供应商的地址、密钥、模型配置
- 一键在配置之间切换，不用手改 JSON / TOML
- 支持 Claude Code、Codex、OpenCode 等多种客户端
- 单个界面里查看和编辑，避免配置写错导致工具起不来

**下载**

- 官网：<https://ccswitch.io>
- GitHub Releases：<https://github.com/farion1231/cc-switch/releases>
- 按系统选 `.exe` / `.dmg` / `.AppImage`

> [!warning] 密钥安全
> 它帮你保管 API Key，配置文件本身不要提交到 Git 仓库，也不要截图外发。

> [!tip] 官方站点只有一个
> 项目 README 明确写了官网是 `ccswitch.io`。搜索时注意别进到同名仿冒站。

## ComfyUI

**定位**：节点式（可视化连线）的 Stable Diffusion / 生成式模型工作流工具。

**能干什么**

- 用节点连线搭建生图流程：加载模型 → 编码提示词 → 采样 → 解码 → 保存
- 支持 SD1.5 / SDXL / Flux 等模型，以及 ControlNet、LoRA、IP-Adapter 等扩展
- 工作流可保存成 JSON 复用、分享
- 支持图生图、局部重绘、放大、视频生成等
- 有 API 模式，可以被其他程序调用

**下载**

- GitHub：<https://github.com/comfyanonymous/ComfyUI>（源码仓库，看安装说明）
- 官网 / 桌面版：<https://www.comfy.org/>
- Windows 一键包：在 Releases 里找 `ComfyUI_windows_portable` 压缩包，解压后双击 `run_nvidia_gpu.bat`

> [!warning] 显存决定能玩多大
> 512×512 的 SD1.5 出图 4GB 显存够用；SDXL 建议 8GB 以上；Flux 这类大模型在 12GB 以下会很吃力。显存不足时优先降分辨率或换 fp8 / GGUF 量化模型。

> [!tip] 模型放哪
> 大模型放 `models/checkpoints`，LoRA 放 `models/loras`，VAE 放 `models/vae`。放错目录节点里就选不到。

## magpie

**定位**：给 Claude Code、Codex、Gemini CLI 等编程 Agent **在一处切换模型**的本地网关工具。官网 <https://usemagpie.ai/zh/>。

**能干什么**

- 供应商（DeepSeek、Kimi、GLM、Qwen、OpenRouter、Ollama，以及已付费的 Claude / ChatGPT 订阅）只需添加一次，之后在 Codex、Claude Code、Gemini CLI 或任何 Agent 里一键选用
- 在本地跑一个网关（默认 `127.0.0.1:3425`），同时会说 OpenAI Chat / Responses、Anthropic Messages、Google Gemini 三种协议，并在两个方向上互转——流式输出、工具调用、推理过程都会转
- 只改写目标 Agent 配置里的那**一项**（`settings.json`、`config.toml`、`config.yaml` 的注释、顺序、缩进都原样保留，写入是原子的），下次会话即生效
- Codex 的 `/model` 列表会直接出现新模型；Claude Code 的 `opus` / `sonnet` / `haiku` 可以映射到 Kimi、GLM 或 GPT；选回「原生模型」时把配置原样恢复
- 支持保存「方案」：一次切换所有 Agent，例如「省钱」「专注」
- 订阅共享：在某个 Agent 里登录过的账号会变成一个供应商，其他 Agent 也能用它的模型，令牌仍按原 Agent 的方式刷新
- 支持按账号做负载与故障转移（智能 / 按顺序 / 轮流 / 用量少的优先），某个账号 429、余额不足或出错时自动换下一个

**下载**

- 官网（中文）：<https://usemagpie.ai/zh/> → 点「下载」，页面会按系统给出对应安装包
- GitHub Releases：<https://github.com/yetone/magpie-releases/releases/latest>
  - Windows：`magpie-windows-amd64.exe`（ARM 机器选 `arm64`）
  - macOS：`magpie-darwin-arm64.dmg`（Intel 选 `amd64`）
  - Linux：`magpie-linux-amd64`
  - 命令行版：`magpie-cli-*`，同页面下载
- 终端脚本安装（macOS / Linux）：页面提供 `curl -fsSL https://usemagpie.ai/install.sh | sh`

> [!warning] 和窗口放大工具同名
> GitHub 上还有一个 `Blinue/Magpie`，那是 Windows 的窗口超分放大工具，和这个完全无关。认准 `usemagpie.ai` 和 `yetone/magpie`。

> [!tip] 先选「原生模型」再逐步替换
> 刚装好先把某个 Agent 设回原生模型，确认网关不影响正常使用，再逐个换成第三方模型。这样出问题时容易判断是网关还是模型本身。

> [!warning] 密钥仍在自己机器上
> 网关跑在本地，API Key 存在本机配置里。别把配置文件（含 `settings.json`、`config.toml`）提交到 Git 仓库。
