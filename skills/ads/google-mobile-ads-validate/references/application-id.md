# 项目中不存在测试应用 ID，格式正确

-   **按以下步骤操作**：
    -   [ ] 在代码库中搜索应用 ID 的键名：

        | 平台 | 键名 |
        | :--- | :--- |
        | **Android** | `InitializationConfig.Builder` 中的 `applicationId` |
        | **iOS** | Info.plist 中的 `GADApplicationIdentifier` 键 |
        | **Unity** | Unity 编辑器设置中的 `adMobAndroidAppId` / `adMobIOSAppId` |

    -   [ ] 如果找到多个应用 ID，逐个评估每个应用 ID。
    -   [ ] 检查是否有应用 ID 使用 Google 官方测试应用 ID：
        -   **AdMob**：`ca-app-pub-3940256099942544~1458002511`
        -   **Ad Manager**：`ca-app-pub-9939518381636264~1458516390`
-   **通过/失败标准**：
    -   **失败**：
        -   任一应用 ID 不匹配正则
            `^(ca-app-pub-[a-zA-Z0-9\-]+)~(.*)$`。
        -   项目中未找到任何应用 ID。
        -   同一平台或构建目标配置了多个应用 ID，且未按 flavor 或环境区分。
    -   **警告**：
        -   找到的任一应用 ID 是 Google 官方测试应用 ID。