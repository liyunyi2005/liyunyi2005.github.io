# npm 版本管理指南

## 方案一：使用 nvm-windows（推荐）

### 1. 安装 nvm-windows
- 下载地址：https://github.com/coreybutler/nvm-windows/releases
- 下载 `nvm-setup.exe` 并安装

### 2. 使用 nvm-windows 管理版本
```powershell
# 查看可用的 Node.js 版本（每个版本都带有对应的 npm）
nvm list available

# 安装特定版本的 Node.js（会自动安装对应的 npm）
nvm install 18.20.0  # 例如安装 Node.js 18.20.0

# 切换到特定版本
nvm use 18.20.0

# 查看已安装的版本
nvm list

# 查看当前使用的版本
nvm current
```

### 3. 不同 Node.js 版本对应的 npm 版本
- Node.js 16.x → npm 8.x
- Node.js 18.x → npm 9.x
- Node.js 20.x → npm 10.x
- Node.js 22.x → npm 10.x

## 方案二：直接安装特定版本的 npm

### 1. 卸载当前 npm（可选）
```powershell
# 以管理员身份运行 PowerShell，然后执行：
npm uninstall -g npm
```

### 2. 安装特定版本的 npm
```powershell
# 以管理员身份运行 PowerShell，然后执行：
npm install -g npm@8.19.4  # 例如安装 npm 8.19.4

# 或者安装其他版本：
# npm install -g npm@9.9.3   # npm 9.x
# npm install -g npm@7.24.2  # npm 7.x
```

### 3. 验证安装
```powershell
npm --version
node --version
```

## 方案三：使用 Corepack（Node.js 内置）

如果你使用的是 Node.js 16.9+ 或更高版本，可以使用 Corepack：

```powershell
# 启用 Corepack
corepack enable

# 这会使用 package.json 中指定的 packageManager
# 你的项目使用 pnpm@9.14.4
```

## 注意事项

1. **PowerShell 执行策略**：如果遇到脚本执行错误，需要以管理员身份运行：
   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

2. **项目使用 pnpm**：你的项目配置了 `packageManager: "pnpm@9.14.4"`，建议使用 pnpm 而不是 npm。

3. **版本兼容性**：确保 npm 版本与 Node.js 版本兼容。

## 推荐操作步骤

### 方法 A：使用管理员权限安装（快速）

1. **以管理员身份运行 PowerShell**：
   - 右键点击"开始"菜单
   - 选择"Windows PowerShell (管理员)"或"终端 (管理员)"

2. **在管理员 PowerShell 中执行**：
   ```powershell
   # 卸载当前 npm
   npm uninstall -g npm
   
   # 安装旧版本 npm（选择一个）
   npm install -g npm@8.19.4    # npm 8.x（推荐，稳定）
   # 或
   npm install -g npm@9.9.3    # npm 9.x
   # 或
   npm install -g npm@7.24.2   # npm 7.x
   
   # 验证安装
   npm --version
   ```

### 方法 B：使用 nvm-windows（推荐，可切换版本）

1. **下载并安装 nvm-windows**：
   - 访问：https://github.com/coreybutler/nvm-windows/releases
   - 下载 `nvm-setup.exe` 并安装

2. **安装旧版本的 Node.js**（会自动安装对应的 npm）：
   ```powershell
   # 查看可用版本
   nvm list available
   
   # 安装 Node.js 18.20.0（对应 npm 9.x）
   nvm install 18.20.0
   
   # 切换到该版本
   nvm use 18.20.0
   
   # 如果需要更旧的 npm，可以单独安装
   npm install -g npm@8.19.4
   ```

### 方法 C：手动替换 npm（高级）

如果上述方法都不行，可以手动下载并替换 npm：

1. 访问 https://github.com/npm/cli/releases
2. 下载对应版本的 npm
3. 替换 `D:\node js\node_modules\npm` 目录

## 当前环境信息

- Node.js 版本：v24.12.0
- npm 版本：11.6.2（位于 D:\node js\）
- 项目使用：pnpm@9.14.4

## 注意事项

⚠️ **重要**：全局安装 npm 需要**管理员权限**，请务必以管理员身份运行 PowerShell！

