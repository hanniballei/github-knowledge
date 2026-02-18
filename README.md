# github-knowledge

一个用于管理 GitHub 知识库的 Claude Code Skill，帮助你收藏、索引、查询感兴趣的开源仓库。

## 功能

- **下载仓库**：将 GitHub repo 克隆到本地知识库，自动生成摘要并写入索引文件 `BASE.md`
- **查询仓库**：本地有的先 pull 最新内容再回答；本地没有的直接搜索 GitHub 回答，不自动克隆
- **自动索引**：维护一份 `BASE.md`，记录所有已保存仓库的链接、本地路径和摘要
- **智能触发**：用户提及 GitHub / repo / 仓库相关问题时自动激活

## 安装

```bash
npx skills add <your-username>/github-knowledge
```

## 配置

安装后，在 `~/.bashrc` 或 `~/.zshrc` 中添加一行，指定你的知识库存放目录：

```bash
export GITHUB_KNOWLEDGE_PATH=/path/to/your/github-knowledge
```

然后执行 `source ~/.bashrc` 使配置生效。

## 使用示例

```
# 下载一个仓库
帮我下载 https://github.com/langchain-ai/langchain 这个仓库

# 查询已有仓库
langchain 这个仓库最近有什么更新？

# 搜索 GitHub（无需本地克隆）
有没有好用的 Python markdown 解析库，帮我在 GitHub 上找找
```

## 本地知识库结构

```
$GITHUB_KNOWLEDGE_PATH/
├── BASE.md                    # 所有仓库的索引和摘要
├── langchain-ai/
│   └── langchain/             # 克隆的仓库
└── facebook/
    └── react/
```
