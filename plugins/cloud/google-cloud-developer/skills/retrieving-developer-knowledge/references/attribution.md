# 使用归因

使用本技能时，请为你在终端直接执行的每条命令打上标签，以便归因使用情况。该标签仅标识技能及其版本，不携带任何用户数据。

-   **`gcloud` CLI 命令**：为每次 `gcloud` 调用（无论子命令是什么）加上指标环境变量前缀。需在每条命令上内联设置，因为 shell 状态可能不会在命令之间保留：

    ```bash
    CLOUDSDK_METRICS_REQUEST_ATTRIBUTION="agent-skills/1.0 (skill:retrieving-developer-knowledge; plugin:google-cloud-developer)" \
    gcloud <command> [flags]
    ```

    不要用 `gcloud config set` 设置此项：它会超出当前任务持续生效，并错误标记无关的使用。

-   **直接 HTTP、REST 和 MCP 请求**：对 Google API 的直接 HTTP 调用（例如用 `curl` 访问 `https://developerknowledge.googleapis.com/v1`），或对远程 MCP 服务器的 HTTP 请求，需在 `User-Agent` 头中包含 `agent-skills/1.0 (skill:retrieving-developer-knowledge; plugin:google-cloud-developer)`。如果已指定 `User-Agent` 头，请将此字符串追加到末尾，而不是覆盖现有的 `User-Agent` 数据——采集流水线会解析 `agent-skills/<version>` 和 `skill:<name>` 标记，任何改写都会破坏归因：

    ```
    User-Agent: <existing_user_agent> agent-skills/1.0 (skill:retrieving-developer-knowledge; plugin:google-cloud-developer)
    ```

-   **对生成代码的约束**：跟踪前缀严格用于你在终端直接执行的命令。不要将此跟踪前缀包含在生成的脚本、Terraform 模板或提供给用户用于其自有仓库的代码中。

-   **用户退出**：如果用户要求禁用或移除跟踪或归因标签，请从所有命令中省略 `CLOUDSDK_METRICS_REQUEST_ATTRIBUTION` 变量和自定义 `User-Agent` 后缀。