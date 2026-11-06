# gcloud CLI 用法

本文档提供在本地和自动化环境中安装、授权和配置 Google Cloud SDK（`gcloud` CLI）的参考信息。

## 安装

如果执行环境中未安装 `gcloud` 二进制文件，请参阅权威的 [Google Cloud CLI 安装指南](https://docs.cloud.google.com/sdk/docs/install-sdk.md.txt)，获取各平台的安装说明（Linux、macOS、Windows、包管理器和容器镜像）。

> [!CAUTION]
> **安装 `gcloud` CLI 前必须获得用户明确同意：**
> 绝不可自主安装 Google Cloud CLI（`gcloud` CLI）或其他 `gcloud` CLI 组件（`gcloud components install` / `gcloud components update`）。
>
> 在执行任何安装命令或包管理器脚本之前：
> 1. 告知用户 `gcloud` CLI（或所需组件）尚未安装，并概述你计划使用的安装方式。
> 2. 请求用户明确同意：`"我已准备好在你的机器上安装 Google Cloud CLI（gcloud）。是否继续？"`
> 3. **关键**：本轮中智能体**不得**执行任何 `gcloud` 安装或组件安装命令。你必须发出同意询问，并**严格停止**，等待用户明确确认。

### 组件管理

`gcloud components` 命令组管理可选的 CLI 组件（如附加工具、模拟器和语言运行时）：

-   **列出可用组件：**

    ```bash
    gcloud components list
    ```

-   **安装组件：**

    ```bash
    gcloud components install {component_id} --quiet
    ```

-   **更新所有已安装组件：**

    ```bash
    gcloud components update --quiet
    ```

*（注意：如果 `gcloud` 是通过 APT 或 DNF 等系统包管理器安装的，请使用系统包管理器安装组件，而非 `gcloud components install`。）*

## 授权与认证

根据运行环境对 CLI 进行 Google Cloud 认证：

-   **用户账号（交互式）：**

    ```bash
    gcloud auth login
    ```

    按浏览器提示登录并授予访问权限。

-   **用户账号（无头流程）：**

    适用于无可用浏览器的环境（容器、远程 SSH）：

    ```bash
    gcloud auth login --no-browser
    ```

    复制生成的 URL，在另一台机器上打开以完成登录，然后将授权码粘贴回终端。

-   **应用默认凭据（ADC）：**

    为客户端库和本地应用配置凭据：

    ```bash
    gcloud auth application-default login
    ```

    在无头环境中追加 `--no-browser`。

-   **服务账号密钥（无头自动化）：**

    ```bash
    gcloud auth activate-service-account --key-file=path/to/key.json
    ```

    *安全提示：限制 JSON 密钥的文件权限，或优先使用工作负载身份联合 / 模拟。*

-   **服务账号模拟（开发与智能体首选）：**

    允许用户身份临时采用服务账号身份，无需存储长期私钥文件：

    ```bash
    gcloud config set auth/impersonate_service_account {service_account_email}
    ```

    需要对目标服务账号具有 `roles/iam.serviceAccountTokenCreator` 角色。这可强制最小权限，并确保以目标身份进行可审计的访问。

-   **工作负载身份联合：**

    适用于 CI/CD 和外部计算环境（GitHub Actions、AWS、本地部署），使用联合令牌认证，无需管理服务账号密钥。参见 [Authorizing the gcloud CLI](https://docs.cloud.google.com/sdk/docs/authorizing.md.txt)。

## 本地配置管理

`gcloud config` 命令组管理本地配置设置、配置文件和默认属性。

### 命名配置

配置允许维护多组隔离的属性（如 dev、staging、prod）：

-   **创建新配置：**

    ```bash
    gcloud config configurations create {config_name}
    ```

-   **列出已有配置：**

    ```bash
    gcloud config configurations list
    ```

-   **激活配置：**

    ```bash
    gcloud config configurations activate {config_name}
    ```

### 设置常用属性

属性为 `gcloud` 各次调用的标志设置默认值：

-   **设置活动项目：**

    ```bash
    gcloud config set core/project {project_id}
    ```

-   **设置默认计算区域和可用区：**

    ```bash
    gcloud config set compute/region {region}
    gcloud config set compute/zone {zone}
    ```

-   **查看所有活动配置属性：**

    ```bash
    gcloud config list
    ```