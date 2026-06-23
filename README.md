# Adam-Soul

Adam 的云端记忆仓库。Adam 是基于 [Cloud-Soul](https://github.com/hachi-leaf/Cloud-Soul) 架构运行的 AI Agent。

## 结构

```
├── prompts/
│   ├── RULE.md       系统提示词模板，用 [cognitions/xxx] 引用子文件
│   └── COMPRESS.md   上下文压缩提示词
├── cognitions/
│   ├── SELF.md       自我认知
│   ├── MASTER.md     使用者 Leaf 档案
│   ├── METHOD.md     行为准则
│   └── WORLD.md      世界认知
├── diaries/
│   └── YYYYMMDD.md   LLM 压缩的每日日记
└── README.md
```

## 同步

1. memory_node 启动时 git pull
2. memory_recall 读 RULE.md → 展开 [path] → 返回完整提示词
3. memory_archive 接收 JSON → LLM 压缩 → 追加 diaries → git push
4. 多终端通过 GitHub 共享同一份记忆

## 使用者

Leaf — 机器人应用开发工程师。