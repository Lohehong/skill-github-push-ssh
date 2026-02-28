---
name: github-push-ssh
description: "使用 SSH 方式推送 Git 仓库到 GitHub 的完整指南。解决 SSH key 生成、配置、ssh-agent 问题等常见问题。"
author: "天依 (Tianyi)"
source: "internal"
created: "2026-03-01"
tags: ["github", "git", "ssh", "push", "deployment"]
metadata: {"openclaw": {"emoji": "🔐", "requires": {"bins": ["git", "ssh", "ssh-keygen"]}}}
---

# GitHub SSH 推送 Skill

> **🏷️ Skill 来源：** 天依实战经验总结  
> **👤 作者：** 天依 (Tianyi)  
> **📅 创建时间：** 2026-03-01  
> **🎯 用途：** 使用 SSH 方式推送 Git 仓库到 GitHub

## 📖 问题背景

**典型场景：**
- 本地有多个 Git 仓库需要推送到 GitHub
- 没有配置 SSH key 或 SSH key 未添加到 GitHub
- ssh-agent 未启动或 key 未加载
- 多个仓库需要批量推送

**常见问题：**
```
❌ git@github.com: Permission denied (publickey).
❌ Could not read from remote repository.
❌ Could not open a connection to your authentication agent.
❌ remote origin already exists.
```

---

## 🚀 完整解决方案

### 步骤 1：生成 SSH Key

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/github_ed25519 -N ""
```

**参数说明：**
- `-t ed25519` : 使用 ed25519 算法（推荐）
- `-C "email"` : 添加注释（通常是邮箱）
- `-f ~/.ssh/...` : 指定保存路径
- `-N ""` : 空 passphrase（方便自动化）

---

### 步骤 2：获取公钥内容

```bash
cat ~/.ssh/github_ed25519.pub
```

复制整行内容（从 `ssh-ed25519` 开始到邮箱结束）

---

### 步骤 3：添加到 GitHub

1. 访问：https://github.com/settings/keys
2. 点击："New SSH key"
3. Title：填写易识别的名称
4. Key type：选择 `Authentication Key`
5. Key：粘贴公钥
6. 点击："Add SSH key"

---

### 步骤 4：测试 SSH 连接

```bash
ssh -T git@github.com
```

**成功输出：**
```
Hi Lohehong! You've successfully authenticated, but GitHub does not provide shell access.
```

---

### 步骤 5：启动 ssh-agent 并添加 key

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519
```

**重要：** `eval "$(ssh-agent -s)"` 必须在 `ssh-add` 之前执行

---

### 步骤 6：配置 remote 并推送

**新仓库：**
```bash
git remote add origin git@github.com:Username/repo-name.git
git branch -M main
git push -u origin main
```

**已有 remote：**
```bash
git remote set-url origin git@github.com:Username/repo-name.git
git branch -M main
git push -u origin main
```

**不确定是否有 remote：**
```bash
git remote add origin git@github.com:Username/repo-name.git 2>/dev/null || git remote set-url origin git@github.com:Username/repo-name.git
git branch -M main 2>/dev/null
git push -u origin main
```

---

## 🔧 批量推送脚本

```bash
#!/bin/bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519

declare -A REPOS=(
    ["~/.openclaw/skills/self-improving-agent"]="skill-self-improvement"
    ["~/.openclaw/skills/micro-tool-development"]="skill-micro-tool-development"
    ["~/.openclaw/skills/tavily-search"]="skill-tavily-search"
)

for dir in "${!REPOS[@]}"; do
    repo="${REPOS[$dir]}"
    echo "Pushing $repo..."
    cd "$dir" || continue
    git remote add origin git@github.com:Lohehong/$repo.git 2>/dev/null || git remote set-url origin git@github.com:Lohehong/$repo.git
    git branch -M main 2>/dev/null
    git push -u origin main
done
```

---

## 🐛 常见问题

### Permission denied (publickey)

```bash
# 1. 确认 key 已添加到 GitHub
cat ~/.ssh/github_ed25519.pub

# 2. 启动 ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519

# 3. 测试
ssh -T git@github.com
```

### Could not open a connection to your authentication agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519
```

### remote origin already exists

```bash
git remote set-url origin git@github.com:Username/repo.git
```

---

## 📋 检查清单

推送前检查：
- [ ] SSH key 已生成
- [ ] 公钥已添加到 GitHub
- [ ] SSH 连接测试通过
- [ ] ssh-agent 已启动
- [ ] key 已添加到 agent
- [ ] 仓库已在 GitHub 创建
- [ ] remote URL 配置正确

---

_最后更新：2026-03-01 | 天依实战总结_

---

## 📝 实战案例：批量维护 README

### 场景

推送 5 个 Git 仓库到 GitHub 后，需要为每个仓库添加 README.md 并推送。

### 步骤

#### 1. 为每个仓库创建 README

```bash
# self-improvement
cd ~/.openclaw/skills/self-improving-agent
cat > README.md << 'EOFREADME'
# 项目标题
...
EOFREADME

git add README.md
git commit -m "docs: 添加 README.md"
```

#### 2. 批量推送

```bash
# 启动 ssh-agent（只需一次）
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_ed25519

# 推送所有仓库
cd ~/.openclaw/skills/self-improving-agent && git push
cd ~/.openclaw/skills/micro-tool-development && git push
cd ~/.openclaw/skills/tavily-search && git push
cd ~/.openclaw/skills/github-push-ssh && git push
cd ~/.openclaw/workspace && git push
```

### 最佳实践

1. **README 内容模板**
   - 项目简介
   - 特性列表
   - 快速开始
   - 目录结构
   - 使用示例
   - 相关链接
   - 作者信息

2. **提交信息规范**
   ```
   docs: 添加 README.md
   
   📝 内容包括:
   - 项目简介和特性
   - 快速开始指南
   - 使用示例
   - 相关链接
   
   Signed-off-by: 天依 (Tianyi)
   ```

3. **批量操作技巧**
   - 先启动 ssh-agent（只需一次）
   - 使用 `&&` 连接命令
   - 每个仓库单独 commit 和 push

### 本次推送结果

| 仓库 | README 行数 | 状态 |
|------|-----------|------|
| skill-self-improvement | 92 行 | ✅ 已推送 |
| skill-micro-tool-development | 100 行 | ✅ 已推送 |
| skill-tavily-search | 113 行 | ✅ 已推送 |
| skill-github-push-ssh | 130 行 | ✅ 已推送 |
| openclaw-workspace-tianyi | 116 行 | ✅ 已推送 |

**总计：** 5 个仓库，551 行文档

---

_最后更新：2026-03-01 | 添加批量维护 README 实战案例_
