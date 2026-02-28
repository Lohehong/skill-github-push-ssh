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

---

## 🐛 实战问题：批量推送 README 遇到的问题

### 问题背景

2026-03-01 为 5 个已推送的 skill 仓库批量添加 README.md 时遇到的一系列问题。

---

### 问题 1：ssh-agent 未启动导致推送失败

**现象：**
```bash
cd ~/.openclaw/skills/self-improving-agent
git add README.md && git commit -m "docs: 添加 README" && git push

# 输出：
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

**原因：**
- 每个 `exec` 命令是独立的 shell 会话
- `eval "$(ssh-agent -s)"` 启动的 agent 只在当前会话有效
- 后续的 `git push` 在新的 shell 会话中，ssh-agent 已失效

**错误做法：**
```bash
# ❌ 这样不行 - agent 只在第一个命令有效
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519
cd repo1 && git push  # 这个会话已经有 agent
cd repo2 && git push  # ❌ 新会话，agent 失效
```

**正确做法：**
```bash
# ✅ 方式 1：每个推送都启动 agent
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519 && cd repo1 && git push
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519 && cd repo2 && git push

# ✅ 方式 2：使用分号保持在同一会话
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519; \
cd repo1 && git push; \
cd repo2 && git push; \
cd repo3 && git push

# ✅ 方式 3：使用后台 agent（推荐）
ssh-add ~/.ssh/github_ed25519 2>/dev/null || {
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/github_ed25519
}
# 然后推送所有仓库
```

**解决方案：**
每次推送前都执行 `eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519`

---

### 问题 2：批量操作时的命令组织

**问题：**
需要为 5 个仓库分别：
1. 创建 README.md
2. git add
3. git commit
4. git push

**低效做法：**
```bash
# 手动一个一个是，重复 5 次
cd repo1 && cat > README.md && git add . && git commit && git push
cd repo2 && cat > README.md && git add . && git commit && git push
...
```

**高效做法：**
```bash
# 使用循环
for repo in repo1 repo2 repo3 repo4 repo5; do
    cd ~/.openclaw/skills/$repo
    cat > README.md << 'EOF'
    # README 内容
    EOF
    git add README.md
    git commit -m "docs: 添加 README.md"
    eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519 && git push
done
```

**本次实际使用的方法：**
```bash
# 为每个仓库单独执行（便于调试）
cd repo1 && cat > README.md << 'EOF'...
git add README.md && git commit -m "..." && eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519 && git push

cd repo2 && cat > README.md << 'EOF'...
git add README.md && git commit -m "..." && eval "$(ssh-agent -s)" && ssh-add ~/.ssh/github_ed25519 && git push
```

---

### 问题 3：README 内容一致性

**问题：**
5 个仓库的 README 需要保持风格一致，但内容各有侧重。

**解决方案：**
使用统一的模板结构：

```markdown
# 项目标题 🎯

> 一句话简介

## 📖 简介

项目描述

## ✨ 特性

- 特性 1
- 特性 2
- 特性 3

## 🚀 快速开始

步骤说明

## 📁 目录结构

```
项目/
├── 文件 1
└── 文件 2
```

## 📚 文档

- 文档链接

## 🔗 相关链接

- 相关资源

## 👤 作者

作者信息

## 📄 许可证

许可证信息
```

**本次 5 个仓库的 README 都遵循了这个结构。**

---

### 问题 4：提交信息规范

**问题：**
批量提交时，提交信息需要规范且包含足够信息。

**解决方案：**
使用统一的提交信息格式：

```
docs: 添加 README.md

📝 内容包括:
- 项目简介和特性
- 快速开始指南
- 目录结构说明
- 相关链接

Signed-off-by: 天依 (Tianyi)
```

**优点：**
- 清晰的类型前缀（docs:）
- emoji 增加可读性（📝）
- 列表说明包含内容
- 签名表明作者

---

### 问题 5：推送顺序和依赖

**问题：**
某些仓库之间有依赖关系（如 github-push-ssh skill 记录了推送经验），需要先推送基础仓库。

**本次推送顺序：**
1. skill-self-improvement ✅
2. skill-micro-tool-development ✅
3. skill-tavily-search ✅
4. skill-github-push-ssh ✅（记录推送经验）
5. openclaw-workspace-tianyi ✅（最后，因为要链接到其他 skill）

**最佳实践：**
- 先推送基础 skill
- 再推送依赖其他 skill 的仓库
- 最后推送 workspace（可能链接到所有 skill）

---

## 📊 本次批量操作统计

### 推送结果

| 仓库 | README 行数 | 提交哈希 | 推送状态 |
|------|-----------|---------|---------|
| skill-self-improvement | 92 行 | dcd0625 | ✅ |
| skill-micro-tool-development | 100 行 | a7c796a | ✅ |
| skill-tavily-search | 113 行 | cab3eed | ✅ |
| skill-github-push-ssh | 130 行 | 1997927 | ✅ |
| openclaw-workspace-tianyi | 116 行 | e949243 | ✅ |

**总计：** 5 个仓库，551 行文档，5 次推送

### 遇到的问题

| 问题 | 次数 | 解决方案 |
|------|------|---------|
| ssh-agent 未启动 | 5 次 | 每次推送前都启动 |
| Permission denied | 5 次 | 正确启动 agent 后解决 |
| 命令组织复杂 | 多次 | 使用分号保持会话 |

### 学到的经验

1. **ssh-agent 是会话级的** - 每个新 shell 需要重新启动
2. **批量操作要规划顺序** - 先基础后依赖
3. **统一模板提高效率** - README 结构一致
4. **规范提交信息** - 便于后续查阅
5. **及时记录经验** - 推送到 skill 中永久保存

---

## ✅ 检查清单（批量推送 README）

批量为多个仓库添加 README 时：

- [ ] 规划 README 模板结构
- [ ] 准备各仓库的特定内容
- [ ] 确定推送顺序（基础→依赖）
- [ ] 为每个仓库创建 README
- [ ] 使用规范的提交信息
- [ ] 每次推送前启动 ssh-agent
- [ ] 验证推送成功
- [ ] 记录经验到 skill

---

_最后更新：2026-03-01 | 添加批量推送 README 实战问题总结_
