# Windows构建指南

## 🚀 使用GitHub Actions自动构建

### 方法1：手动触发Windows构建

1. **访问GitHub仓库**
   - 打开 https://github.com/meimingqi222/opcode
   - 点击 `Actions` 标签页

2. **选择Windows构建工作流**
   - 在左侧找到 `Build Windows`
   - 点击 `Run workflow` 按钮
   - 选择 `main` 分支
   - 点击绿色的 `Run workflow` 按钮

3. **等待构建完成**
   - 构建大约需要10-15分钟
   - 你可以点击运行中的工作流查看实时日志

4. **下载构建产物**
   - 构建完成后，滚动到页面底部
   - 在 `Artifacts` 部分找到 `windows-x86_64`
   - 点击下载zip文件

### 方法2：推送到main分支触发构建

每次推送到main分支都会自动触发所有平台的构建：

```bash
git add .
git commit -m "your changes"
git push origin main
```

### 方法3：创建Release触发完整构建

创建Git标签会触发完整的release构建流程：

```bash
# 创建新标签
git tag v0.3.0
git push origin v0.3.0
```

这将触发完整的release流程，包括：
- Linux构建 (.deb, .AppImage)
- macOS构建 (.dmg, .app.tar.gz) 
- Windows构建 (.msi, .exe)
- 自动创建GitHub Release

## 📦 Windows构建产物

构建完成后你将获得：

- `opcode.msi` - Windows MSI安装包
- `opcode.exe` - NSIS安装程序
- `opcode.exe` - 独立可执行文件
- `checksums.txt` - SHA256校验和

## 🔧 本地验证（可选）

如果你想在本地验证构建配置：

```powershell
# 检查Rust工具链
rustc --version
cargo --version

# 检查配置文件语法
cd src-tauri
cargo check --quiet
```

## 🌟 构建状态监控

你可以在GitHub仓库的README中添加构建状态徽章：

```markdown
[![Windows Build](https://github.com/meimingqi222/opcode/workflows/Build%20Windows/badge.svg)](https://github.com/meimingqi222/opcode/actions)
```

## 📋 故障排除

如果构建失败：

1. 检查GitHub Actions日志中的错误信息
2. 确保所有配置文件语法正确
3. 验证依赖项版本兼容性
4. 查看WARP.md中的故障排除部分

## 🎯 推荐的工作流程

1. **开发阶段**: 使用手动触发进行测试构建
2. **测试阶段**: 推送到main分支进行CI构建
3. **发布阶段**: 创建Git标签进行完整release构建

这样你就可以在不配置本地Windows开发环境的情况下，获得完全可用的Windows版本！
