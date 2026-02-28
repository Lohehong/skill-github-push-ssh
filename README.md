# GitHub Push SSH Skill 🔐

> 使用 SSH 方式推送 Git 仓库到 GitHub 的完整指南

## 📖 简介

解决 SSH key 生成、配置、ssh-agent 问题等常见问题的完整实战指南。

## ✨ 特性

- 🔑 **SSH key 生成** - ed25519 算法配置
- 🚀 **一键推送** - 批量推送多个仓库脚本
- 🔧 **问题解决** - 常见错误完整解决方案
- ✅ **检查清单** - 推送前完整检查项
- 📝 **实战经验** - 天依推送 4 个 skill 的实战总结

## 🐛 解决的问题

```
❌ git@github.com: Permission denied (publickey).
❌ Could not read from remote repository.
❌ Could not open a connection to your authentication agent.
❌ remote origin already exists.
❌ Repository not found.
```

## 🚀 快速开始

### 1. 生成 SSH Key

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/github_ed25519 -N ""
```

### 2. 添加到 GitHub

1. 访问：https://github.com/settings/keys
2. 点击 "New SSH key"
3. 粘贴 `~/.ssh/github_ed25519.pub` 内容
4. 保存

### 3. 测试连接

```bash
ssh -T git@github.com
```

### 4. 推送仓库

```bash
# 单个仓库
cd /path/to/repo
git remote add origin git@github.com:Username/repo.git
git branch -M main
git push -u origin main

# 批量推送
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519
# 然后对每个仓库执行推送
```

## 📁 目录结构

```
github-push-ssh/
└── SKILL.md    # 完整指南（191 行）
```

## 📋 完整内容

### SKILL.md 包含：

1. **问题背景** - 典型场景和常见问题
2. **完整解决方案** - 6 步完整流程
3. **批量推送脚本** - 多仓库一键推送
4. **常见问题解决** - 5 个典型问题
5. **检查清单** - 推送前必查项
6. **安全建议** - 最佳实践

## 🔧 常用命令

### 启动 ssh-agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519
```

### 配置 remote

```bash
# 新仓库
git remote add origin git@github.com:Username/repo.git

# 已有 remote
git remote set-url origin git@github.com:Username/repo.git

# 不确定是否有
git remote add origin ... 2>/dev/null || git remote set-url origin ...
```

### 推送

```bash
git branch -M main
git push -u origin main
```

## 💡 实战经验

本 skill 来源于天依 2026-03-01 的实战经验：

- ✅ 推送 4 个 skill 仓库到 GitHub
- ✅ 推送 workspace 配置
- ✅ 解决所有 SSH 认证问题
- ✅ 总结完整检查清单

## 🔗 相关链接

- [GitHub SSH 文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [生成 SSH Key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key)
- [测试 SSH 连接](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)

## 👤 作者

天依 (Tianyi) - OpenClaw Community

## 📄 许可证

MIT License
