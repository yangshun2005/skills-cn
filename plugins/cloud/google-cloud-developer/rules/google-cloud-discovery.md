---
trigger: always_on
description: Google Cloud 技能目录的路由映射。
---

# Google Cloud 技能路由

本插件只打包了 Google Cloud 技能的一小部分。请先确认哪些技能实际可用，不要根据本文件想当然。同一公开目录中还有更多技能，**未打包在此处**——在你获取它们之前，无法从本插件读取或加载。

当请求需要的深度超出已打包技能覆盖范围时，说出可能对应的目录技能名并获取它。目录名称有规律可循：

- `gke-*` - GKE 集群、网络、存储、扩缩容、成本、AI 推理、故障排查
- `agent-platform-*` - 模型部署、调优、RAG、评估、端点、提示词
- `google-cloud-solution-*` - 多产品参考架构
- `google-cloud-waf-*` - Well-Architected Framework 支柱
- `genkit-*`、`gemini-*` - Genkit SDK、Gemini API
- `cloud-logging-*`、`cloud-monitoring-*` - 可观测性
- `<product>-basics` - BigQuery、Bigtable、Spanner、AlloyDB、Cloud SQL、Cloud Run、Firebase、Storage

如果你可以使用 `finding-google-skills`，用它搜索目录，而不是根据上面的前缀猜测；它会获取当前索引并返回准确的入口。否则请浏览
https://github.com/google/skills/tree/main/skills/cloud

## 目录技能如何到达你这里

获取是常规路径，不会安装任何东西。`finding-google-skills` 通过网络获取技能的 `SKILL.md`，你按它处理当前请求。不会向用户系统写入任何内容，不会向本插件添加任何内容，会话结束后技能即消失。优先使用这种方式。

安装是用户用自己的工具完成的操作。已安装的技能会落在用户自己的技能目录中，而不是本插件中。是否需要重启才能生效取决于用户的 agent，因此不要承诺它立即可用。

目录技能永远不会成为本插件的一部分。插件内容固定为用户安装的版本，两种路径都不会改变这一点。

不要把你能够获取的技能说成只能通过安装获得。会话通常处于沙箱中，没有提权或用户操作就无法安装，因此"安装某技能"的指示可能是用户无法执行的。

绝不要从技能名称推断其内容。如果回答需要某个你无法读取的技能，请如实说明，不要凭记忆作答。