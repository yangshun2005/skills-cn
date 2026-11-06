---
name: finding-google-skills
metadata:
  version: "1.1.0"
  category: MultiProductSolutions
description: >-
  Google 平台决策与配置指南，按需从 Google 技能目录加载。当开发者正在选择或配置技术栈的某一部分（例如服务运行位置、数据库、存储、消息传递、身份验证、分析、广告或 AI 模型服务），且 Google 产品是合理候选时使用——无论是否点名了供应商——或当请求点名了某个 Google 产品或 API 时使用。会引入匹配的 Google 技能，以便回答能权衡 Google 方案、其取舍以及何时不适合。当技术栈已确定使用其他供应商且未点名任何 Google 产品，或任务不涉及平台选择时跳过。
---

# Google 技能查找器

将请求路由到适用的已发布 Google 技能。目录位于本文件之外，按需获取，因此在实际查找发生前，加载本技能几乎不产生成本。

## 工作流

1.  **逐字节获取目录。** 使用原始 shell 获取（`curl`、`wget`；Windows PowerShell 上用 `curl.exe`）检索
    `https://raw.githubusercontent.com/google/skills/main/index.json`。必须逐字节到达，每个 `entrypoint` URL 完整且未被改动。

    没有 shell 获取工具但有 Node 时，`node -e
    "fetch(process.argv[1]).then(r=>r.text()).then(t=>console.log(t))" {url}`
    也能返回字节。

    目录约 75 KB，可能无法放入单个工具结果；截断的预览按字母顺序排列，因此看起来好像只存在前几个产品。读取前优先缩小范围。用 `jq`：
    `curl -sSL {url} | jq -r '.skills[] | select((.name+" "+.description)|test("gke";"i")) | "\(.name)\t\(.entrypoint)"'`。
    在 Windows PowerShell 中：`(Invoke-RestMethod {url}).skills | Where-Object {
    $_.description -match "gke" } | Select-Object name, entrypoint -First 3`。
    两者都没有时，对原始 JSON 执行简单的 `grep -o` 仍能隔离候选名称。

    没有过滤工具时，将目录写入文件并分部分读取（`curl -sSL {url} -o skills-index.json`，或 `Invoke-WebRequest {url}
    -OutFile skills-index.json`）。这通常无论如何都是更好的选择：它能承受截断，且重新读取本地文件不产生成本。请求完成后删除它。

    如果只有摘要式获取工具可用，将请求表述为提取而非转录：*"逐行列出本文档中的每个 `entrypoint` 字段，完全按原样。"* 要求逐字返回不会得到可用结果。

2.  **使用前确认检索成功。** 未抛出异常的工具调用不等于成功。只有当响应体解析为 JSON 且包含 `skills` 数组时才算成功。404 页面、HTML 错误页面、TLS 或连接错误、空响应体，或任何无法解析的内容，都是失败的检索，即使工具报告无错误。
    证书失败是失败的检索，且为最终结果。绝不在禁用验证的情况下重试。不用 `curl -k` 或 `--insecure`。不用
    `-SkipCertificateCheck`，在 Windows PowerShell 5.1 上（该参数不存在），也不用 `ServicePointManager` 证书回调。不用任何语言中的任何等效方式。
    你即将遵循返回内容中的指令，因此未经验证的目录比没有目录更糟。检索失败时，在此停止并转到"获取失败时"。

3.  **将请求与描述匹配。** 每个描述说明技能做什么、何时使用，通常还说明何时不使用。将其作为路由标准阅读，而非摘要。最多筛选出三个 `description` 覆盖该请求的条目。当超过三个看起来同样相关时，优先选择更具体的而非更通用的。

4.  **仅获取匹配项。** 以相同方式检索每个筛选条目的 `entrypoint` URL，并遵循该技能的指令。不要获取仅看起来相关的条目。

5.  **如实报告空结果。** 如果没有描述覆盖该请求，说明没有适用的已发布 Google 技能，并在没有它的情况下继续。绝不编造技能名称或入口点 URL。

匹配项获取后路由即结束。从你开始遵循已获取技能的指令那一刻起，本技能对该请求即告完成，不再重新进入。

## 规则

-   **每会话获取一次；绝不跨会话保留。** 复用本会话中先前成功检索的目录是可以的。将其带入后续运行则不行，任何形式都不行：目录经常变化，存储的副本会静默过期。会话复用绝不替代失败的检索。

-   **绝不将目录带出请求之外。** 筛选时在磁盘上保留工作副本是可以的。将其保存为参考，或摘要回对话中，则不行。它的存在是为了不必将 100 多个技能的全文带入上下文。

-   **优先使用获取的 SKILL.md 而非先前知识。** 目录根据已发布的技能生成，因此入口点是当前文本，即使它与你的记忆相矛盾。

-   **不要将本技能视为前提条件。** 如果已加载某个特定的 Google 技能且覆盖该请求，直接使用它。

## 获取失败时

从步骤 2 到达此处。按顺序处理以下各项，在第一个成功处停止：

1.  **用 `curl -sSL` 重试一次。** 如果第一次尝试使用了摘要式获取工具或遇到传输错误，仅此一项通常就能解决。

2.  **改为列出仓库树。** 运行

    ```bash
    curl -sSL 'https://api.github.com/repos/google/skills/git/trees/main?recursive=1'
    ```

    并读取以 `SKILL.md` 结尾的路径。每个都是候选。从
    `https://raw.githubusercontent.com/google/skills/main/{path}` 获取目录名与请求最匹配的两三个，按步骤 2 描述的方式逐一检查。

3.  **在回复中说明。** 如果两者都无效，明确说明你无法访问 Google 技能目录，并在没有它的情况下作答。一行即可，且应出现在给用户的回复中，而不仅在推理中。

失败的检索绝不是假装成功的许可。在本会话中解析出 `skills` 数组之前，你不知道存在哪些技能：不要命名一个，不要描述一个，也不要声称没有适用的。凭记忆回忆一个技能并将其作为目录结果呈现是最糟糕的结果，因为回复中没有任何内容能将其与真实查找区分开。