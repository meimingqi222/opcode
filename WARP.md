# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## 项目概述

opcode是一个基于Tauri 2的桌面应用，为Claude Code提供GUI界面。这是一个混合架构项目：
- 前端：React 18 + TypeScript + Vite 6
- 后端：Rust with Tauri 2
- UI框架：Tailwind CSS v4 + shadcn/ui
- 数据库：SQLite (via rusqlite)
- 包管理器：Bun

## 核心开发命令

### 基本开发
```bash
# 开发模式 (带热重载)
bun run tauri dev

# 前端开发 (仅限前端)
bun run dev

# 生产构建
bun run tauri build

# 类型检查
bunx tsc --noEmit

# 代码检查
bun run check  # TypeScript + Rust
```

### Rust后端
```bash
# 运行Rust测试
cd src-tauri && cargo test

# 代码格式化
cd src-tauri && cargo fmt

# Clippy代码检查
cd src-tauri && cargo clippy

# 仅构建后端
cd src-tauri && cargo build
```

### 特定平台构建
```bash
# macOS通用二进制 (Intel + Apple Silicon)
bun run tauri build --target universal-apple-darwin

# Windows x86_64构建
bun run tauri build --target x86_64-pc-windows-msvc

# Linux x86_64构建
bun run tauri build --target x86_64-unknown-linux-gnu

# 调试构建 (更快编译)
bun run tauri build --debug

# 特定格式构建
bun run build:dmg  # macOS DMG
```

### Windows特定命令
```powershell
# PowerShell 中的类型检查
bunx tsc --noEmit

# Windows MSI安装包构建
bun run tauri build --bundles msi

# Windows NSIS安装包构建
bun run tauri build --bundles nsis

# 查看构建结果
ls src-tauri\target\release\bundle\msi\*.msi
ls src-tauri\target\release\bundle\nsis\*.exe
```

## 项目架构

### 目录结构
- `src/` - React前端代码
  - `components/` - UI组件
  - `lib/` - API客户端和工具函数
  - `hooks/` - React自定义钩子
  - `stores/` - Zustand状态管理
  - `services/` - 业务逻辑服务
  - `types/` - TypeScript类型定义
- `src-tauri/src/` - Rust后端代码
  - `commands/` - Tauri命令处理器
  - `checkpoint/` - 时间线管理模块
  - `process/` - 进程管理
- `cc_agents/` - 预构建AI智能体
- `public/` - 静态资源

### 核心技术栈详解

**状态管理**: 使用Zustand进行全局状态管理，主要store包括：
- `sessionStore.ts` - Claude会话状态
- `agentStore.ts` - CC智能体状态

**数据持久化**: SQLite数据库，存储：
- 智能体配置
- 会话历史  
- 使用统计
- 系统设置

**进程管理**: Rust进程注册表管理Claude Code CLI实例和智能体执行

### 关键功能模块

1. **项目和会话管理**: 管理`~/.claude/projects/`中的项目
2. **CC智能体系统**: 自定义AI智能体，支持后台执行
3. **时间线和检查点**: 会话版本控制系统
4. **使用分析**: API使用情况和成本跟踪
5. **MCP服务器管理**: Model Context Protocol服务器集成

## 依赖关系

### 系统依赖
- Claude Code CLI (必须在PATH中可用)
- Rust 1.70.0+
- Bun (最新版本)

### 平台特定依赖

**Linux (Ubuntu/Debian)**:
```bash
sudo apt install -y libwebkit2gtk-4.1-dev libgtk-3-dev \
  libayatana-appindicator3-dev librsvg2-dev libssl-dev \
  libxdo-dev libsoup-3.0-dev libjavascriptcoregtk-4.1-dev
```

**macOS**:
```bash
xcode-select --install
```

**Windows**:
```powershell
# 安装 Microsoft C++ Build Tools
# 从 https://visualstudio.microsoft.com/visual-cpp-build-tools/ 下载

# 安装 WebView2 (通常在 Windows 11 中预安装)
# 从 https://developer.microsoft.com/microsoft-edge/webview2/ 下载

# 安装 Windows SDK (可选，但推荐)
winget install "Windows SDK"

# 检查 MSVC 工具链安装
where cl.exe
where link.exe
```

## CI/CD工作流

### 测试流程 (.github/workflows/build-test.yml)
- 支持多平台构建测试：Linux, Linux ARM64, Windows, macOS
- 自动缓存Rust和Bun依赖
- PR时自动状态评论

### 发布流程 (.github/workflows/release.yml)
- Git标签触发自动发布
- 生成跨平台安装包：
  - macOS: `.dmg`, `.app.tar.gz` (Universal)
  - Linux: `.AppImage`, `.deb`
  - Windows: `.msi`, `.exe` (NSIS installer), `.exe` (standalone)
- 自动生成SHA256校验和
- GitHub Release自动创建

### 分离的构建工作流
- `build-linux.yml` - Linux构建
- `build-macos.yml` - macOS构建  
- `build-windows.yml` - Windows构建
- 使用工作流复用避免重复

## 开发环境配置

### 必需配置
1. 确保Claude Code CLI已安装：`claude --version`
2. Rust工具链：通过rustup安装
3. Bun包管理器：`curl -fsSL https://bun.sh/install | bash`

### 首次设置
```bash
git clone <repository>
cd opcode
bun install
bun run tauri dev
```

## 特殊注意事项

### Tauri配置
- 使用CSP (Content Security Policy) 增强安全性
- macOS启用vibrancy效果和圆角
- 支持无边框透明窗口
- 文件系统权限限制在`$HOME/**`

### Windows 特定配置
- 支持 MSI 和 NSIS 安装包格式
- 代码签名支持 (CI/CD 中使用 WINDOWS_CERTIFICATE secrets)
- Windows Defender SmartScreen 兼容性
- 自动生成应用程序快捷方式

#### 本地 Windows 开发配置
```powershell
# 设置环境变量用于代码签名（可选）
$env:TAURI_SIGNING_PRIVATE_KEY = "your-certificate-base64"
$env:TAURI_SIGNING_PRIVATE_KEY_PASSWORD = "your-certificate-password"

# 或者使用 Windows 证书存储库
# 用于从证书存储库中自动选择证书
$env:TAURI_WINDOWS_CERTIFICATE_THUMBPRINT = "your-certificate-thumbprint"
```

### 代码标准
- TypeScript: 严格模式，导出函数需JSDoc
- Rust: 遵循标准约定，显式处理Result类型
- 安全要求: 验证所有前端输入，从不记录敏感数据

### 测试策略
- Rust: `cargo test`
- 手动测试应用功能
- CI中跨平台构建验证

## 智能体系统

opcode支持创建和管理Claude Code智能体：
- 存储在`cc_agents/`目录
- 使用`.opcode.json`格式
- 支持GitHub导入/导出
- 后台进程执行

可用智能体包括Git提交机器人、安全扫描器、单元测试生成器等。

## 调试和故障排除

### 常见问题
1. **"claude command not found"** - 确保 Claude Code CLI 在 PATH 中
2. **Linux webkit 错误** - 安装 webkit2gtk 开发包
3. **内存不足构建失败** - 使用 `cargo build -j 2` 限制并行作业
4. **Windows: "MSVC not found"** - 安装 Visual Studio Build Tools 和 C++ 支持
5. **Windows: "WebView2 错误"** - 下载并安装 Microsoft Edge WebView2 Runtime
6. **Windows: "编译器错误"** - 确保使用正确的 Visual Studio 组件

### 构建验证
```bash
# Linux/macOS 验证
./src-tauri/target/release/opcode

# Windows 验证 (PowerShell)
.\src-tauri\target\release\opcode.exe

# Windows 验证 (Command Prompt)
src-tauri\target\release\opcode.exe
```

### Windows 特定调试
```powershell
# 检查 Rust 工具链
rustc --version
cargo --version

# 检查 MSVC 编译器
where cl

# 检查 Windows SDK
reg query "HKLM\SOFTWARE\Microsoft\Windows Kits\Installed Roots"

# 检查 WebView2 安装
reg query "HKLM\SOFTWARE\WOW6432Node\Microsoft\EdgeUpdate\Clients\{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}"

# 清理构建缓存
cargo clean
rm -rf target/

# 重新构建
bun install
bun run tauri build
```

