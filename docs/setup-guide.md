# Claude Code Skills 管理指南

## 背景

Claude Code 的个人技能存放在 `~/.claude/skills/` 目录下。该目录本身不适合直接做 git 仓库（`~/.claude/` 下有凭证、缓存等自动生成文件）。

解决方案：将技能文件放在独立 git 仓库中，通过符号链接映射到 `~/.claude/skills/`。

## 仓库结构

```
claude-skills/
├── team-dev/SKILL.md         # 多智能体研发团队
├── team-review/SKILL.md      # 多智能体代码审查团队
├── team-arch/SKILL.md        # 多智能体架构分析团队
├── team-research/SKILL.md    # 多智能体通用调研团队
└── docs/plans/               # 设计文档
```

## 新机器部署（macOS / Linux 通用）

### 1. 克隆仓库

```bash
# 选择你的代码目录，以下示例用 ~/Code/git
cd ~/Code/git
git clone git@github.com:ketor/claude-skills.git
```

### 2. 确保 skills 目录存在

```bash
mkdir -p ~/.claude/skills
```

### 3. 创建符号链接

```bash
REPO=~/Code/git/claude-skills

ln -s "$REPO/team-dev"      ~/.claude/skills/team-dev
ln -s "$REPO/team-review"   ~/.claude/skills/team-review
ln -s "$REPO/team-arch"     ~/.claude/skills/team-arch
ln -s "$REPO/team-research" ~/.claude/skills/team-research
ln -s "$REPO/docs"          ~/.claude/skills/docs
```

### 4. 验证

```bash
ls -la ~/.claude/skills/
# 应该看到所有条目都是 -> 指向仓库的符号链接
```

## 日常使用

### 编辑技能后提交

```bash
cd ~/Code/git/claude-skills
git add -A
git commit -m "描述你的修改"
git push
```

### 另一台机器同步

```bash
cd ~/Code/git/claude-skills
git pull
# 符号链接自动生效，无需其他操作
```

### 新增技能

```bash
# 1. 在仓库中创建
mkdir ~/Code/git/claude-skills/new-skill
# 编辑 SKILL.md ...

# 2. 创建符号链接
ln -s ~/Code/git/claude-skills/new-skill ~/.claude/skills/new-skill

# 3. 提交
cd ~/Code/git/claude-skills
git add new-skill
git commit -m "feat: add new-skill"
git push
```

## 注意事项

- `~/.claude/` 下的其他文件（`settings.json`、`.credentials.json`、`cache/` 等）由 Claude Code 自动管理，不要纳入版本控制
- 符号链接在 macOS 和 Linux 上行为一致，`ln -s` 命令通用
- 如果 `~/.claude/skills/` 下已有同名目录，先删除再创建链接
