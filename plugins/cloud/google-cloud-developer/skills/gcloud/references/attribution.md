# 使用归因

**必读**：使用本技能执行任何 `gcloud`、REST API 或 MCP 命令前，请遵循以下使用归因说明。为你在终端直接执行的每条命令打上标签，以便归因使用情况。标签仅标识技能及其版本，不携带任何用户数据。

-   **技能优先级（多技能会话）**：`gcloud` 是基础 CLI 技能，常与产品专属技能预捆绑或同时加载。如果当前任务还加载或激活了更具体的产品、服务或领域技能（例如 `cloud-run-basics`、`cloud-build-basics`、`bigquery-basics` 或 `gke-basics`），**务必**使用该具体技能的 `skill:<name>` 归因标签，而非 `skill:gcloud`。仅当所执行命令没有适用的产品专属技能时，才使用 `skill:gcloud`。

-   **`gcloud` CLI 命令**：无论子命令是什么，每次调用 `gcloud` 都要在命令前加上指标环境变量。由于 shell 状态可能不会在命令间保留，需在每条命令上内联设置：

    ```bash
    CLOUDSDK_METRICS_REQUEST_ATTRIBUTION="agent-skills/1.0 (skill:gcloud; plugin:google-cloud-developer)" \
    gcloud <command> [flags]
    ```

    不要用 `gcloud config set` 设置：它会超出当前任务持续生效，并错误标记无关使用。

-   **直接 HTTP、REST 和 MCP 请求**：直接调用 Google Cloud API 的 HTTP 请求（例如使用 `curl`）或发往远程 MCP 服务器的 HTTP 请求，需在 `User-Agent` 头中包含 `agent-skills/1.0 (skill:gcloud; plugin:google-cloud-developer)`。如果已指定 `User-Agent` 头，请将此字符串追加到末尾，不要覆盖现有 `User-Agent` 数据——采集流水线会解析 `agent-skills/<version>` 和 `skill:<name>` 标记，任何改写都会破坏归因：

    ```
    User-Agent: <existing_user_agent> agent-skills/1.0 (skill:gcloud; plugin:google-cloud-developer)
    ```

-   **生成代码的约束**：跟踪前缀严格仅用于你在终端直接执行的命令。不要在生成的脚本、Terraform 模板或提供给用户用于其自有仓库的代码中包含此跟踪前缀。

-   **用户退出**：如果用户要求禁用或移除跟踪或归因标签，请从所有命令中省略 `CLOUDSDK_METRICS_REQUEST_ATTRIBUTION` 变量和自定义 `User-Agent` 后缀。