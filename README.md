

> **[中文化] skills**
>
> 此项目是 [google/skills](https://github.com/google/skills) 的中文翻译版本。
> - 原项目 Stars: 22761
> - 主语言: Python
> - 许可证: Apache-2.0
> - 翻译日期: 2026-11-07
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

这是 Google 官方维护的 Agent Skills 仓库，收录面向 Google 产品与技术（包括 Google Cloud）的 Agent Skills。它解决的是：让 AI Agent 在涉及 Google Cloud 各类场景（如 IAM 认证、GKE、BigQuery、Gemini API 等）时，能按预置的技能内容执行任务，而不必每次从零描述。仓库本身是技能集合，不是可独立运行的应用。

### 快速上手（最短路径）

1. 确认环境中有 `npx`；若没有，先安装 Node.js（自带 `npm` 和 `npx`）。
2. 执行安装命令：
   ```bash
   npx skills add google/skills
   ```
3. 按提示选择要安装的技能（单个或全部）、要配置的目标 Agent，以及安装范围（项目级或全局）。
4. 后续如需更新已安装技能，执行：
   ```bash
   npx skills update
   ```
   该命令会提供一并移除过时技能的选项。

### 环境要求

- 操作系统：原文未说明
- 硬件：原文未说明
- 依赖：需要 `npx`；若环境没有 `npx`，需安装 Node.js（包含 `npm` 和 `npx`）
- 版本要求：原文未说明
- 国内网络环境提示：安装依赖 Node.js 与 `npx` 在线拉取，网络受限时可能需要配置镜像源或加速下载（原文未说明具体做法）。

### 常见坑与注意事项

- 环境里没有 `npx` 时安装命令无法执行，需先装 Node.js。
- 安装过程是交互式的，需要依次选择技能、目标 Agent 和安装范围，不是一条命令直接装完。
- 仓库包含大量 Google Cloud 相关技能，部分技能可能涉及 Google Cloud 账号与认证（如 IAM 相关技能），但原文未说明具体前置条件。
- 更新时可能提示移除过时技能，注意确认以免误删仍在用的技能。

---



---

# Agent Skills

[![通过 skills.sh 安装](https://img.shields.io/badge/skills.sh-install-green)](https://skills.sh/google/skills)

本仓库包含面向 Google 产品与技术的 [Agent Skills](https://agentskills.io/home)，包括 [Google Cloud](https://cloud.google.com)。

## 安装

```bash
npx skills add google/skills
```

根据提示选择特定技能或全部技能、要配置的目标 Agent，以及安装范围（项目或全局）。

如果环境中没有 `npx`，请安装 [Node.js](https://nodejs.org/en/download)，其中包含 `npm` 和 `npx`。

## 更新技能

要将已安装的技能更新到最新版本，请运行以下命令。

```bash
npx skills update
```

该命令还会提供移除已过时技能的选项。

## 可用技能

<!-- BEGIN SKILLS -->
- **Google Cloud 入门**
  - [**Google Cloud 身份验证与授权 (IAM)**](./skills/cloud/google-cloud-recipe-auth)
  - [**Google Cloud 配方：基础构建器**](./skills/cloud/google-cloud-recipe-foundation-builder)
  - [**Google Cloud 入门引导**](./skills/cloud/google-cloud-recipe-onboarding)
- **多产品解决方案技能**
  - [**Google Cloud 解决方案架构工作流**](./skills/cloud/google-cloud-solution-architecture)
  - [**Agent Gateway 多 Agent 安全**](./skills/cloud/google-cloud-solution-multi-agent-security)
  - [**跨云提供商和数据类型的 Agentic 分析**](./skills/cloud/google-cloud-solution-agentic-analytics-spark-knowledge-catalog)
  - [**无边界开放数据湖仓 Agentic AI 系统**](./skills/cloud/google-cloud-solution-agentic-ai-borderless-data-lakehouse)
  - [**在 Google Cloud 上构建和部署 AI Agent**](./skills/cloud/google-cloud-solution-build-deploy-agents)
  - [**使用 AI Agent 的数据科学工作流解决方案**](./skills/cloud/google-cloud-solution-agentic-ai-data-science-workflow)
  - [**使用 AlloyDB 的动态混合搜索**](./skills/cloud/google-cloud-solution-hybrid-search-alloydb)
  - [**Google 技能查找器**](./skills/developers/finding-google-skills)
  - [**实时双向多模态流式 Agentic AI 解决方案**](./skills/cloud/google-cloud-solution-agentic-ai-bidirectional-streaming)
  - [**将 AI 工作负载迁移到 GKE Inference**](./skills/cloud/google-cloud-solution-guided-gke-ai-migration)
  - [**使用 GKE 和 AlloyDB 的企业搜索 RAG**](./skills/cloud/google-cloud-solution-rag-enterprise-search-gke-sqldb)
  - [**具有严格私有应用层的安全 n 层无服务器 Web 应用**](./skills/cloud/google-cloud-solution-n-tier-serverless-web-app)
- **AI/ML**
  - [**Agent 访问审计**](./skills/cloud/agent-platform-access-audit)
  - [**Agent Platform 告警配置**](./skills/cloud/agent-platform-alert-configuration)
  - [**Agent Platform 端点管理**](./skills/cloud/agent-platform-endpoint-management)
  - [**Agent Platform 评估飞轮技能**](./skills/cloud/agent-platform-eval-flywheel)
  - [**Agent Platform GenAI 推理技能**](./skills/cloud/agent-platform-inference)
  - [**Agent Platform Model Garden 部署技能**](./skills/cloud/agent-platform-deploy)
  - [**Agent Platform 模型注册表管理**](./skills/cloud/agent-platform-model-registry)
  - [**Agent Platform 模型调优**](./skills/cloud/agent-platform-tuning)
  - [**Agent Platform 提示词管理**](./skills/cloud/agent-platform-prompt-management)
  - [**Agent Platform RAG Engine 管理**](./skills/cloud/agent-platform-rag-engine-management)
  - [**Agent Platform 故障排查**](./skills/cloud/agent-platform-troubleshooting)
  - [**Agent Platform 调优管理**](./skills/cloud/agent-platform-tuning-management)
  - [**API Gateway 远程 MCP Server（公开预览版）**](./skills/cloud/api-gateway-mcp-server)
  - [**Apigee 作为 Agent Gateway 服务扩展**](./skills/cloud/apigee-agent-gateway-governance)
  - [**BigQuery AI 与 ML**](./skills/cloud/bigquery-ai-ml)
  - [**Agent Platform 中的 Gemini API**](./skills/cloud/gemini-api)
  - [**Gemini Enterprise Agent Platform - 托管 Agent API 技能**](./skills/cloud/gemini-agents-api)
  - [**Gemini Interactions API 技能**](./skills/cloud/gemini-interactions-api)
  - [**Gemini 平台架构与工程指南**](./skills/cloud/agent-platform-architecture)
  - [**Genkit Dart**](./skills/cloud/developing-genkit-dart)
  - [**Genkit Go**](./skills/cloud/developing-genkit-go)
  - [**Genkit JS**](./skills/cloud/developing-genkit-js)
  - [**Genkit Python**](./skills/cloud/developing-genkit-python)
  - [**LiveAPI 服务技能**](./skills/cloud/gemini-live-api)
  - [**从 AI Studio 中的 Gemini API 迁移到 Agent Platform**](./skills/cloud/agent-platform-migrate-from-ai-studio)
  - [**技能注册表**](./skills/cloud/agent-platform-skill-registry)
  - [**使用 MXLA Hang Analyzer 排查 GKE TPU 多切片挂起问题**](./skills/cloud/gke-ai-troubleshooting-tpu-mxla-hang)
  - [**使用 ML Diagnostics Workload Monitoring 排查 GKE TPU 性能下降问题**](./skills/cloud/gke-ai-troubleshooting-tpu-performance-degradation)
- **基础设施**
  - [**Cloud Network Insights**](./skills/cloud/cloud-network-insights)
  - [**Cloud Storage 基础**](./skills/cloud/google-cloud-storage-basics)
  - [**Cloud Storage 存储桶架构师**](./skills/cloud/google-cloud-storage-bucket-architect)
  - [**Cloud Storage 诊断**](./skills/cloud/google-cloud-storage-diagnostic)
  - [**Cloud Storage FUSE**](./skills/cloud/google-cloud-storage-fuse)
  - [**Compute Engine 未来预留与日历模式可行性**](./skills/cloud/compute-future-reservations)
  - [**Compute Engine 按需预留**](./skills/cloud/compute-on-demand-reservations)
  - [**使用 Application Design Center 设计和部署 GCP 基础设施**](./skills/cloud/application-design-center-design-deploy)
  - [**Filestore 审计技能**](./skills/cloud/google-cloud-filestore-auditing)
  - [**Filestore 自动扩缩**](./skills/cloud/google-cloud-filestore-autoscale)
  - [**Filestore 基于日志的故障排查**](./skills/cloud/google-cloud-filestore-log-troubleshooting)
  - [**Filestore NFS 文件浏览器**](./skills/cloud/google-cloud-filestore-nfs-browser)
  - [**GKE AI/ML 推理**](./skills/cloud/gke-inference)
  - [**GKE 告警配置**](./skills/cloud/gke-alert-configuration)
  - [**GKE 应用接入**](./skills/cloud/gke-app-onboarding)
  - [**GKE 备份与灾难恢复**](./skills/cloud/gke-backup-dr)
  - [**GKE 基础与关键注意事项**](./skills/cloud/gke-basics)
  - [**GKE 批处理与 HPC 工作负载**](./skills/cloud/gke-batch-hpc)
  - [**GKE 集群自动扩缩器**](./skills/cloud/gke-cluster-autoscaler)
  - [**GKE 集群创建**](./skills/cloud/gke-cluster-creation)
  - [**GKE ComputeClasses**](./skills/cloud/gke-compute-classes)
  - [**GKE 黄金基础镜像发现专家**](./skills/cloud/gke-custom-golden-image-discovery)
  - [**GKE 黄金路径配置**](./skills/cloud/gke-golden-path)
  - [**GKE JobSet 中断故障排查**](./skills/cloud/gke-ai-troubleshooting-jobset-interruption)
  - [**GKE 清单生成技能**](./skills/cloud/gke-manifest-generation)
  - [**GKE 多租户**](./skills/cloud/gke-multitenancy)
  - [**GKE 网络**](./skills/cloud/gke-networking)
  - [**GKE 节点 NotReady 故障排查技能**](./skills/cloud/gke-node-notready)
  - [**GKE 生产化技能**](./skills/cloud/gke-productionize)
  - [**GKE 可靠性**](./skills/cloud/gke-reliability)
  - [**GKE 服务网络技能**](./skills/cloud/gke-service-networking)
  - [**GKE 存储**](./skills/cloud/gke-storage)
  - [**GKE 存储故障排查技能**](./skills/cloud/gke-storage-troubleshooting)
  - [**GKE TPU 动态切片监控与管理**](./skills/cloud/gke-ai-troubleshooting-tpu-dynamic-slices-monitoring)
  - [**GKE 升级与维护**](./skills/cloud/gke-upgrades)
  - [**GKE 工作负载扩缩**](./skills/cloud/gke-workload-scaling)
  - [**GKE 工作负载扩缩故障排查技能**](./skills/cloud/gke-workload-scaling-troubleshooting)
  - [**GKE 工作负载故障排查技能**](./skills/cloud/gke-workload-troubleshooting)
  - [**Google Cloud 全球外部应用负载均衡器配置技能**](./skills/cloud/google-cloud-global-frontend-configuration)
  - [**Google Cloud 网络可观测性专家**](./skills/cloud/google-cloud-networking-observability)
  - [**排查因节点池容量泄漏导致的 GKE TPU 预留缺货问题**](./skills/cloud/gke-ai-troubleshooting-tpu-reservation-stockout)
  - [**排查因 MIG 不稳定导致 GKE TPU 切片创建受阻的问题**](./skills/cloud/gke-ai-troubleshooting-tpu-slice-mig-stuck-updating)
  - [**排查卡在 Compute Engine 修复中（`REPAIRING`）的 GKE TPU 虚拟机**](./skills/cloud/gke-ai-troubleshooting-tpu-vm-repair-stuck)
  - [**排查因分区重叠导致的 TPU 切片创建失败**](./skills/cloud/gke-ai-troubleshooting-tpu-partition-overlap)
  - [**排查无响应的 GKE TPU 和 GPU 节点（`NodeStatusUnknown`）**](./skills/cloud/gke-ai-troubleshooting-node-unresponsive-timeout)
- **数据库与分析**
  - [**AlloyDB 基础**](./skills/cloud/alloydb-basics)
  - [**BigFrames（BigQuery DataFrame）基础**](./skills/cloud/bigquery-bigframes)
  - [**BigQuery 资产影响分析**](./skills/cloud/datalineage-bigquery-asset-impact-analysis)
  - [**BigQuery 基础**](./skills/cloud/bigquery-basics)
  - [**BigQuery 迁移服务 (BQMS) SQL 转换器**](./skills/cloud/bigquery-migration-sql-translator)
  - [**BigQuery 可观测性**](./skills/cloud/bigquery-observability)
  - [**BigQuery 优化工作流**](./skills/cloud/bigquery-optimization)
  - [**BigQuery 故障排查**](./skills/cloud/bigquery-troubleshooting)
  - [**BigQuery 用户定义函数 (UDF)**](./skills/cloud/bigquery-udfs)
  - [**Bigtable 基础**](./skills/cloud/bigtable-basics)
  - [**Cloud SQL 基础**](./skills/cloud/cloud-sql-basics)
  - [**数据血缘摘要**](./skills/cloud/datalineage-summary)
  - [**dbt Snowflake 到 BigQuery 转换器**](./skills/cloud/dbt-sf-to-bq-translator)
  - [**GCP 托管 Airflow DAG 编写指南**](./skills/cloud/managed-airflow-dag-authoring)
  - [**Google Cloud 数据库入门技能**](./skills/cloud/cloud-databases-onboarding)
  - [**Managed Service for Apache Airflow（原 Cloud Composer）DAG 故障排查指南**](./skills/cloud/managed-airflow-dag-troubleshooting)
  - [**Managed Service for Apache Airflow（原 Cloud Composer）迁移指南**](./skills/cloud/managed-airflow-migrations)
  - [**Managed Service for Apache Airflow（原 Cloud Composer）建议**](./skills/cloud/managed-airflow-recommendations)
  - [**Spanner 基础**](./skills/cloud/spanner-basics)
- **开发者工具**
  - [**Cloud Build 基础**](./skills/cloud/cloud-build-basics)
  - [**Cloud Operator**](./skills/cloud/cloud-operator)
  - [**Developer Device Platform**](./skills/cloud/developer-device-platform-basics)
  - [**面向 AI Agent 的 gcloud CLI 技能**](./skills/cloud/gcloud)
  - [**Google Agents CLI 入门**](./skills/cloud/google-agents-cli-onboarding)
  - [**Google 开发者知识**](./skills/developers/retrieving-developer-knowledge)
- **应用开发**
  - [**API Gateway OpenAPI 规范编写**](./skills/cloud/api-gateway-openapi-spec-authoring)
  - [**Application Integration 编写**](./skills/cloud/application-integration-authoring)
  - [**Application Integration 基础**](./skills/cloud/application-integration-basics)
  - [**Application Integration 故障排查**](./skills/cloud/application-integration-troubleshooting)
  - [**将遗留代码库迁移到安全 Rust**](./skills/developers/migrate-to-rust)
- **管理工具**
  - [**Cloud Logging FinOps**](./skills/cloud/cloud-logging-finops)
  - [**Cloud Monitoring 图表生成技能（`cloud-monitoring-chart-generation`）**](./skills/cloud/cloud-monitoring-chart-generation)
  - [**Cloud Monitoring ListTimeSeries 请求生成器**](./skills/cloud/cloud-monitoring-list-time-series-request)
  - [**Cloud Monitoring 通知渠道配置**](./skills/cloud/cloud-monitoring-notification-channels-configuration)
  - [**Cloud Monitoring PromQL 生成器**](./skills/cloud/cloud-monitoring-promql-query)
  - [**Cloud Trace 查询技能（`cloud-trace-querying`）**](./skills/cloud/cloud-trace-querying)
  - [**配置 Cloud Monitoring 告警**](./skills/cloud/cloud-monitoring-alert-configuration)
  - [**配置 Cloud Logging**](./skills/cloud/cloud-logging-configuration-basics)
  - [**配置跨项目日志记录**](./skills/cloud/cloud-logging-cross-project-configuration)
  - [**生成 Logging 查询语言查询**](./skills/cloud/cloud-logging-query-generation)
  - [**GKE 成本分析**](./skills/cloud/gke-cost-analysis)
  - [**GKE 成本优化**](./skills/cloud/gke-cost-optimization)
  - [**GKE 可观测性**](./skills/cloud/gke-observability)
  - [**GKE TPU 指标监控指南**](./skills/cloud/gke-ai-troubleshooting-tpu-metrics-monitoring)
  - [**处理 GPU 和 TPU 中断故障排查**](./skills/cloud/gke-ai-troubleshooting-handle-disruption-gpu-tpu)
  - [**指标选择（服务查询与本地关键词过滤）**](./skills/cloud/cloud-monitoring-metric-selection)
  - [**SLO 告警配置设置向导**](./skills/cloud/google-cloud-slo-alert-configuration)
  - [**TPU 连接失败与 VBAR OOM 故障排查**](./skills/cloud/gke-ai-troubleshooting-tpu-vbar-oom)
  - [**Workload Manager 基础**](./skills/cloud/workload-manager-basics)
- **Well-Architected Framework**
  - [**Google Cloud Well-Architected Framework 成本优化支柱技能**](./skills/cloud/google-cloud-waf-cost-optimization)
  - [**Google Cloud Well-Architected Framework 卓越运营支柱技能**](./skills/cloud/google-cloud-waf-operational-excellence)
  - [**Google Cloud Well-Architected Framework 性能优化支柱技能**](./skills/cloud/google-cloud-waf-performance-optimization)
  - [**Google Cloud Well-Architected Framework 可靠性支柱技能**](./skills/cloud/google-cloud-waf-reliability)
  - [**Google Cloud Well-Architected Framework 安全支柱技能**](./skills/cloud/google-cloud-waf-security)
  - [**Google Cloud Well-Architected Framework 可持续性支柱技能**](./skills/cloud/google-cloud-waf-sustainability)
- **安全与身份**
  - [**API Gateway 后端安全与速率限制**](./skills/cloud/api-gateway-backend-security-and-rate-limiting)
  - [**Cloud Armor OWASP 规则升级器（CRS 3.0/3.3 -> 4.22）**](./skills/cloud/cloud-armor-crs-rule-upgrader)
  - [**合规环境评估器技能**](./skills/cloud/assuredworkloads-environment-assessor)
  - [**自定义资源编写与合成技能（`assuredworkloads-frameworks-author`）**](./skills/cloud/assuredworkloads-frameworks-author)
  - [**数据边界框架导航器（`assuredworkloads-frameworks-navigator`）**](./skills/cloud/assuredworkloads