# Obsidian 归档（可选，替代飞书）

如果不使用飞书，可以将学习材料归档到本地的 Obsidian vault，并通过 git 同步到 GitHub。

## 前置条件

1. **Obsidian vault** 已初始化并配置了 git 远程仓库（如 GitHub）
2. `OBSIDIAN_VAULT_PATH` 环境变量已设置，指向你的 vault 路径（默认：`~/Documents/Obsidian Vault`）

## 归档步骤

面试通过后，将材料从 `materials/` 复制到 Obsidian vault：

```bash
VAULT="${OBSIDIAN_VAULT_PATH:-$HOME/Documents/Obsidian Vault}"
TARGET="$VAULT/Daily-Learn/YYYY-MM-DD-<slug>"
mkdir -p "$TARGET"

# 复制5个核心文件
cp materials/YYYY-MM-DD-<slug>/lesson.md "$TARGET/"
cp materials/YYYY-MM-DD-<slug>/exam.md "$TARGET/"
cp materials/YYYY-MM-DD-<slug>/exam-answers.md "$TARGET/"
cp materials/YYYY-MM-DD-<slug>/interview.md "$TARGET/"
cp materials/YYYY-MM-DD-<slug>/review.md "$TARGET/"
```

在 `$VAULT/Daily-Learn/_index.md` 中追加一条索引记录（文件不存在则创建）：

```markdown
| YYYY-MM-DD | [<Keyword>](Daily-Learn/YYYY-MM-DD-<slug>/lesson.md) | <category> | ✅ |
```

提交并推送到 GitHub：

```bash
cd "$VAULT"
git add -A
git commit -m "📚 Daily-Learn: YYYY-MM-DD - <Keyword>"
git push
```

## 目录结构

```
Documents/Obsidian Vault/
└── Daily-Learn/
    ├── _index.md              ← 所有完成主题的目录索引
    ├── 2026-07-06-transformer/
    │   ├── lesson.md
    │   ├── exam.md
    │   ├── exam-answers.md
    │   ├── interview.md
    │   └── review.md
    └── ...
```

## 可选的定时同步

如果平时也在 Obsidian 里记笔记，建议设置定时任务自动 push：

```bash
cron: 0 9,18 * * *
task: cd vault && git add -A && git commit -m "sync" && git push
```
