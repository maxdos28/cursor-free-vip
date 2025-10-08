# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Cursor Free VIP 是一个基于 Python 的工具，用于管理 Cursor IDE 的配置和账户。它提供了重置机器 ID、注册账户、绕过版本检查以及跨 Windows、macOS 和 Linux 平台管理身份验证的功能。

**重要提示**：这是一个教育工具。代码库会操作 Cursor IDE 的配置文件和身份验证机制。在使用此代码时，应专注于理解现有行为，而不是创建可能违反服务条款的新功能。

## 开发命令

### 运行应用程序

```bash
# 直接使用 Python 运行
python main.py

# 或使用可执行文件（构建后）
./dist/CursorFreeVIP_<version>_<os>
```

### 构建

```bash
# 使用构建脚本
python build.py

# 或使用特定平台的脚本
./build.sh          # Linux/macOS
./build.bat         # Windows
./build.mac.command # macOS（双击运行）

# 手动使用 PyInstaller 构建
pyinstaller build.spec
```

构建过程：
- 从 `.env` 文件读取版本号（VERSION 变量）
- 使用 PyInstaller 和 `build.spec` 配置
- 输出到 `dist/` 目录，命名格式：`CursorFreeVIP_{version}_{os_type}`
- 打包包含 locales、quit_cursor.py、utils.py 和 .env

### 安装和依赖

```bash
# 安装依赖
pip install -r requirements.txt

# 主要依赖：
# - DrissionPage (>=4.0.0) - 浏览器自动化
# - selenium - Web 自动化
# - colorama - 终端颜色
# - psutil - 进程管理
# - requests - HTTP 请求
# - faker - 生成假数据
# - python-dotenv - 环境变量
```

## 架构

### 核心入口

- **main.py**：主菜单系统，支持多语言（通过 `locales/` 目录的 JSON 文件支持 15 种语言）
  - 使用 `Translator` 类实现国际化
  - 在 Windows 上检查管理员权限
  - 提供所有操作的交互式菜单

### 配置系统

- **config.py**：集中式配置管理
  - 在 `~/Documents/.cursor-free-vip/config.ini` 创建配置
  - 管理 Windows/macOS/Linux 的路径
  - 处理浏览器路径（Chrome、Edge、Firefox、Brave、Opera、Opera GX）
  - 自动化延迟的时间配置
  - OAuth 设置和 TempMailPlus 集成

- **配置文件位置**：`Documents/.cursor-free-vip/config.ini`

### 核心模块

#### 身份验证和注册
- **cursor_auth.py**：管理 Cursor 的 SQLite 数据库身份验证
  - 连接到 `state.vscdb`（特定平台路径）
  - 在 ItemTable 中更新认证令牌

- **oauth_auth.py**：OAuth 身份验证处理器
  - 浏览器配置文件选择
  - 从 cookies 提取令牌
  - 通过 DrissionPage 支持多种浏览器

- **new_signup.py**：账户注册自动化
  - 使用 DrissionPage 进行浏览器自动化
  - 使用类人延迟填写表单
  - 邮箱验证流程

- **cursor_register_manual.py**：使用自定义邮箱手动注册
- **manual_custom_auth.py**：自定义身份验证流程
- **get_user_token.py**：从浏览器 cookies 提取令牌

#### 重置和绕过操作
- **totally_reset_cursor.py**：完全重置 Cursor
  - 重置机器 ID
  - 清除身份验证
  - 修改 `workbench.desktop.main.js` 以绕过检查
  - 修改前创建备份

- **reset_machine_manual.py**：手动重置机器 ID
- **restore_machine_id.py**：从备份恢复机器 ID
- **bypass_version.py**：绕过 Cursor 版本检查
- **bypass_token_limit.py**：绕过令牌使用限制
- **disable_auto_update.py**：禁用 Cursor 自动更新
  - 从 `product.json` 删除更新 URL
  - 删除更新程序目录

#### 工具模块
- **utils.py**：跨平台工具函数
  - `get_user_documents_path()`：获取文档文件夹路径
  - `get_default_browser_path()`：浏览器可执行文件路径
  - `get_default_driver_path()`：WebDriver 路径
  - `get_linux_cursor_path()`：Linux Cursor 安装路径
  - `get_random_wait_time()`：类人延迟时间

- **quit_cursor.py**：跨平台关闭 Cursor 进程
- **account_manager.py**：保存/管理账户信息到 `cursor_accounts.txt`
- **check_user_authorized.py**：验证用户授权状态
- **cursor_acc_info.py**：显示账户信息
- **delete_cursor_google.py**：删除 Google 账户关联

### 平台特定路径

工具为每个操作系统管理不同的路径：

**Windows**：
- Storage：`%APPDATA%\Cursor\User\globalStorage\storage.json`
- SQLite：`%APPDATA%\Cursor\User\globalStorage\state.vscdb`
- Machine ID：`%APPDATA%\Cursor\machineId`
- App：`%LOCALAPPDATA%\Programs\Cursor\resources\app`

**macOS**：
- Storage：`~/Library/Application Support/Cursor/User/globalStorage/storage.json`
- SQLite：`~/Library/Application Support/Cursor/User/globalStorage/state.vscdb`
- Machine ID：`~/Library/Application Support/Cursor/machineId`
- App：`/Applications/Cursor.app/Contents/Resources/app`

**Linux**：
- Storage：`~/.config/Cursor/User/globalStorage/storage.json`
- SQLite：`~/.config/Cursor/User/globalStorage/state.vscdb`
- Machine ID：`~/.config/cursor/machineid`
- App：`/opt/Cursor/resources/app`（或 `get_linux_cursor_path()` 检查的其他位置）

### 国际化

- **locales/**：15 种语言的 JSON 文件（ar、bg、de、en、es、fr、it、ja、nl、pt、ru、tr、vi、zh_cn、zh_tw）
- **fill_missing_translations.py**：使用翻译 API 自动翻译缺失的键
- 翻译键使用点号表示法：`menu.title`、`register.filling_form` 等

### 构建和分发

- **build.py**：带进度指示器的构建自动化
- **build.spec**：PyInstaller 规范文件
  - 单文件可执行程序
  - 包含 locales 和辅助脚本
  - 特定平台命名

- **GitHub Actions**（`.github/workflows/build.yml`）：
  - 为 Windows、macOS（Intel/ARM64）、Linux（x64/ARM64）构建
  - 创建带 SHA256 校验和的发布版本
  - 从 CHANGELOG.md 提取发布说明
  - 通过 workflow_dispatch 手动触发

### 安装脚本

- **scripts/install.sh**：自动下载并运行最新版本（Linux/macOS）
- **scripts/install.ps1**：自动下载并运行最新版本（Windows）
- **scripts/reset.ps1**：Windows 重置脚本

## 重要注意事项

1. **管理员权限**：许多操作需要管理员/root 权限来修改 Cursor 文件
2. **必须关闭 Cursor**：在运行重置/修改操作之前，务必关闭 Cursor
3. **备份**：工具在修改关键文件之前会创建备份（`.old` 后缀）
4. **浏览器自动化**：使用 DrissionPage（基于 Chromium）进行 OAuth 流程 - 需要安装浏览器
5. **配置持久化**：配置存储在文档文件夹中，跨运行保留
6. **版本管理**：版本存储在 `.env` 文件中，由构建过程和更新检查使用

## 代码模式

- **错误处理**：使用 colorama 进行广泛的 try-catch 和彩色控制台输出
- **表情符号常量**：每个模块定义 EMOJI 字典用于视觉反馈
- **翻译器模式**：通过函数调用传递 `translator` 对象以实现国际化
- **平台检测**：使用 `sys.platform` 或 `platform.system()` 进行特定操作系统的逻辑
- **配置访问**：使用 `get_config(translator)` 加载配置
- **类人延迟**：使用 `get_random_wait_time(config, 'timing_key')` 进行自动化

## 测试

不存在自动化测试套件。需要跨平台手动测试。

## 版本信息

当前版本：1.11.03（来自 `.env`）

详细的版本历史和更改请参见 `CHANGELOG.md`。
