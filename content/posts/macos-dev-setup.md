---
title: "macOS 开发环境配置指南"
date: 2026-03-08T14:30:00+08:00
draft: false
tags: ["macos", "开发环境", "homebrew", "效率工具"]
categories: ["技术"]
description: "新 Mac 到手后的开发环境配置清单，包含常用工具和效率软件"
---

新 Mac 到手，第一时间配置开发环境。记录一下我的配置流程，方便下次换机。

## 基础工具

### Homebrew

macOS 必备的包管理器：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Git

```bash
brew install git

# 配置
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
```

### iTerm2 + Oh My Zsh

```bash
brew install --cask iterm2
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

推荐插件：

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

## 开发工具

### 编辑器

```bash
brew install --cask visual-studio-code
brew install --cask cursor  # AI 编辑器
```

VS Code 推荐插件：
- Python
- Prettier
- GitLens
- Error Lens
- Todo Tree

### 终端工具

```bash
brew install fzf      # 模糊搜索
brew install ripgrep  # 快速搜索
brew install fd       # 文件查找
brew install bat      # 语法高亮 cat
brew install eza      # 现代化 ls
brew install starship # 提示符美化
brew install zoxide   # 智能 cd
```

### 容器和虚拟化

```bash
brew install docker
brew install --cask docker
```

### 数据库工具

```bash
brew install --cask tableplus
```

## 编程语言

### Node.js

```bash
brew install nvm
nvm install 20
nvm use 20
```

### Python

```bash
brew install pyenv
pyenv install 3.12
pyenv global 3.12

pip install pipx
pipx install poetry
pipx install black
pipx install ruff
```

### Go

```bash
brew install go
```

### Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## 效率工具

```bash
brew install --cask raycast      # 启动器
brew install --cask shottr       # 截图
brew install --cask cleanshot    # 专业截图
brew install --cask rectangle    # 窗口管理
brew install --cask hiddenbar    # 隐藏菜单栏图标
brew install --cask monitorcontrol # 外接显示器亮度控制
```

## 浏览器

```bash
brew install --cask google-chrome
brew install --cask firefox
brew install --cask arc          # 新浏览器，值得尝试
```

## 其他推荐

- **思源黑体**: `brew install --cask font-sf-pro`
- **ClashX**: 网络工具
- **Bob**: OCR 翻译
- **Sip**: 取色工具
- **Ice**: 菜单栏管理

## 备份配置

使用 Mackup 备份应用配置：

```bash
brew install mackup
mackup backup  # 备份到 iCloud/Dropbox
mackup restore # 新机器恢复
```

## 完整 Brewfile

导出当前安装：

```bash
brew bundle dump --describe
```

在新机器上一键安装：

```bash
brew bundle
```

## 总结

以上是我常用的 macOS 开发环境配置。根据项目需求，还会安装不同的工具和 SDK。

有什么推荐的工具？欢迎分享！
