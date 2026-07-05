# 阶段 5：归档

触发：面试通过后

1. 生成 `interview-transcript.md`（模板见 `references/file-templates.md`）
2. 更新 `COMPLETED.md`、`state.json` → `completed`、`topics.json` → `completed`
3. 归档到外部知识库（二选一）：

**选项 A：飞书归档（默认）**
- 按 `references/feishu-sync.md` 上传全部 6 个文件到飞书

**选项 B：Obsidian 归档（备选）**
- 按 `references/obsidian-archive.md` 复制到本地 Obsidian vault 并推送到 GitHub
- 设置 `ARCHIVE_METHOD=obsidian` 环境变量以启用
