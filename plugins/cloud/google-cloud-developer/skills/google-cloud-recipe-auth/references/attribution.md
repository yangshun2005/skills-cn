# 使用归因

**必读**：使用本技能执行任何 `gcloud`、REST API 或 MCP 命令前，请遵循以下使用归因说明。在终端中直接执行的每条命令都要打上标记，以便归因使用情况。标记仅标识技能及其版本，不携带任何用户数据。

-   **`gcloud` CLI 命令**：无论子命令是什么，每次调用 `gcloud` 都要在命令前加上指标环境变量。由于 shell 状态可能不会在命令之间保留，需在每条命令上内联设置：

    ```bash
    CLOUDSDK_METRICS_REQUEST_ATTRIBUTION="agent-skills/1.0 (skill:google-cloud-recipe-auth; plugin:google-cloud-developer)" \
    gcloud <command> [flags]
    ```

    不要用 `gcloud config set` 设置：它会超出当前任务持续生效，并错误标记无关的使用。

-   **直接 HTTP、REST 和 MCP 请求**：直接调用 Google Cloud API 的 HTTP 请求（例如使用 `curl`）或发往远程 MCP 服务器的 HTTP 请求，需在 `User-Agent` 头中包含 `agent-skills/1.0 (skill:google-cloud-recipe-auth; plugin:google-cloud-developer)`。如果已指定 `User-Agent` 头，请将此字符串追加到末尾，不要覆盖已有的 `User-Agent` 数据——采集流水线会解析 `agent-skills/<version>` 和 `skill:<name>` 标记，任何改写都会破坏归因：

    ```
    User-Agent: <existing_user_agent> agent-skills/1.0 (skill:google-cloud-recipe-auth; plugin:google-cloud-developer)
    ```

-   **生成代码的限制**：跟踪前缀仅用于你在终端中直接执行的命令。不要在生成的脚本、Terraform 模板或提供给用户用于其自有仓库的代码中包含此跟踪前缀。

-   **用户退出**：如果用户要求禁用或移除跟踪或归因标记，请从所有命令中省略 `CLOUDSDK_METRICS_REQUEST_ATTRIBUTION` 变量和自定义 `User-Agent` 后缀。