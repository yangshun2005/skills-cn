# Data Cloud 插件

本目录以 **git 子模块**形式引入 [Google Data Cloud](https://cloud.google.com/data-cloud) 插件，使其可通过中央 `google/skills` 仓库被发现和安装。每个子模块固定到其上游插件仓库的特定发布标签，该上游仓库仍是该插件技能和 MCP 服务器定义的**唯一可信来源**。

这些插件为各自产品的常见用户旅程打包了产品专属的**技能**和（适用时）**MCP 服务器**。此集合与 [`GoogleCloudPlatform/data-cloud-plugins`](https://github.com/GoogleCloudPlatform/data-cloud-plugins) 的布局保持一致。

## 安装

### Antigravity CLI（`agy`）

Antigravity CLI 直接从仓库路径安装插件。将 `agy` 指向所需插件：

```bash
agy plugin install https://github.com/google/skills/plugins/cloud/data-cloud/alloydb
agy plugin install https://github.com/google/skills/plugins/cloud/data-cloud/spanner
```

> [!NOTE]
> 此处使用子模块，是因为 Antigravity CLI 尚不支持市场清单（Claude Code 和 Codex 支持）。一旦 `agy` 支持市场清单，这些子模块即可弃用，改用共享清单。

Claude Code 和 Codex 请改用本仓库根目录的市场清单安装。

## 包含的插件

每个插件固定到所示发布标签。要将工作树更新到固定版本，请在仓库根目录运行 `git submodule update --init`。

| 产品 | 仓库 | 版本 | 描述 |
| :--- | :--- | :--- | :--- |
| **AlloyDB for PostgreSQL** | [alloydb-plugin](https://github.com/GoogleCloudPlatform/alloydb-plugin) | `0.2.0` | 创建、连接 AlloyDB for PostgreSQL 数据库并与其数据交互。 |
| **AlloyDB Omni** | [alloydb-omni-plugin](https://github.com/GoogleCloudPlatform/alloydb-omni-plugin) | `0.2.2` | 创建、连接 AlloyDB Omni 数据库并与其数据交互。 |
| **BigQuery Data Analytics** | [bigquery-data-analytics-plugin](https://github.com/GoogleCloudPlatform/bigquery-data-analytics-plugin) | `0.2.5` | 连接 BigQuery 数据集和数据、执行查询并生成数据洞察。 |
| **Cloud SQL for MySQL** | [cloud-sql-mysql-plugin](https://github.com/GoogleCloudPlatform/cloud-sql-mysql-plugin) | `0.2.0` | 连接 Cloud SQL for MySQL 数据库并与其数据交互。 |
| **Cloud SQL for PostgreSQL** | [cloud-sql-postgresql-plugin](https://github.com/GoogleCloudPlatform/cloud-sql-postgresql-plugin) | `0.4.0` | 创建、连接 Cloud SQL for PostgreSQL 数据库并与其数据交互。 |
| **Cloud SQL for SQL Server** | [cloud-sql-sqlserver-plugin](https://github.com/GoogleCloudPlatform/cloud-sql-sqlserver-plugin) | `0.2.0` | 连接 Cloud SQL for SQL Server 数据库并与其交互。 |
| **Data Agent Kit** | [data-agent-kit-plugin](https://github.com/GoogleCloudPlatform/data-agent-kit-plugin) | `1.0.0` | 面向 Google Cloud 数据工程师和数据库从业者的专用技能套件——设计数据流水线、用 dbt 转换数据、编写 Spark/BigQuery notebook，并编排端到端工作流。 |
| **Dataproc** | [dataproc-plugin](https://github.com/GoogleCloudPlatform/dataproc-plugin) | `0.1.0` | 管理 Dataproc 集群和作业。 |
| **DB Context Engineering Agent** | [db-context-enrichment](https://github.com/GoogleCloudPlatform/db-context-enrichment) | `v0.7.2` | 编写和维护 QueryData / Conversational Analytics API 上下文集，让 NL→SQL 规划器了解你的 Schema 词汇和黄金查询形态。 |
| **Firestore** | [firestore-native-plugin](https://github.com/GoogleCloudPlatform/firestore-native-plugin) | `0.3.4` | 连接 Cloud Firestore 并与其交互。 |
| **Google Cloud Storage** | [google-cloud-storage-plugin](https://github.com/GoogleCloudPlatform/google-cloud-storage-plugin) | `3.0.1` | 为编码代理提供的经过审核的 Google Cloud Storage 技能。 |
| **Knowledge Catalog** | [knowledge-catalog-plugin](https://github.com/GoogleCloudPlatform/knowledge-catalog-plugin) | `0.5.4` | 连接 Knowledge Catalog（原 Dataplex），以发现、管理、监控和治理数据平台中的数据与 AI 资产。 |
| **Looker** | [looker-plugin](https://github.com/GoogleCloudPlatform/looker-plugin) | `0.3.11` | 连接 Looker 并使用 LookML 与数据交互。 |
| **Oracle Database** | [oracledb-plugin](https://github.com/GoogleCloudPlatform/oracledb-plugin) | `0.2.7` | 连接 Oracle 数据库、执行查询并与其数据交互。 |
| **Spanner** | [spanner-plugin](https://github.com/GoogleCloudPlatform/spanner-plugin) | `0.3.6` | 使用自然语言连接 Spanner 数据并与其交互。 |

## 更新固定版本

每个子模块跟踪的标签记录在顶层 `.gitmodules`（`branch = <version>`）中。要将插件升级到更新的发布版本，请将其子模块更新到新标签并提交指针变更：

```bash
cd plugins/cloud/data-cloud/<plugin>
git fetch --tags
git checkout <new-version>
cd -
git config -f .gitmodules submodule.<plugin>.branch <new-version>
git add .gitmodules plugins/cloud/data-cloud/<plugin>
git commit -m "feat(<plugin>): bump to <new-version>"
```