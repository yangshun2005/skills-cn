---
name: google-mobile-ads-validate
description: >-
  验证项目的 Google Mobile Ads (GMA) SDK 集成，适用于 iOS、Android 或 Unity 项目。用于对集成 GMA SDK 的应用进行完整的发布前审计，或验证任意单项 GMA SDK 集成检查，例如验证广告单元 ID 和广告格式、SKAdNetwork ID、中介适配器 SDK 版本兼容性，或广告预加载。
metadata:
  version: "1.0.0"
  category: GoogleAds
---

# 验证 Google Mobile Ads SDK 集成

验证项目的 Google Mobile Ads (GMA) SDK 集成，可执行完整审计或仅执行指定的检查。

-   **完整审计**：如果用户请求常规验证或完整审计，评估所有检查清单项。
-   **指定检查**：如果用户只要求验证某个特定方面（例如广告预加载），仅评估相关检查项，不运行整个检查清单。

## 评分规则

对每项检查，应用以下状态之一：

-   **通过**：未满足警告、失败或 N/A 的任何条件。
-   **警告**、**失败**或 **N/A**：满足各状态所述条件。

## 验证检查清单

阅读每项要执行的检查对应的参考指南：

-   项目中无测试应用 ID，格式正确：
    `references/application-id.md`
-   项目中无测试广告单元，格式正确：
    `references/ad-units.md`
-   已实现所有 Google SKAdNetwork ID：
    `references/google-skadnetwork-ids.md`
-   中介适配器兼容性：
    `references/mediation-adapter-compatibility.md`
-   广告预加载验证检查：`references/ad-preloading.md`

## 最终输出

按以下格式生成 Markdown 报告。**仅**包含实际检查过的项目结果。

| 检查 | 状态 | 结果 | 后续步骤 |
| :--- | :---: | :--- | :--- |
| 项目中无测试应用 ID，格式正确 | {{status_1}} | {{findings_1}} | {{next_steps_1}} |
| 项目中无测试广告单元，格式正确 | {{status_2}} | {{findings_2}} | {{next_steps_2}} |
| 已实现所有 Google SKAdNetwork ID | {{status_3}} | {{findings_3}} | {{next_steps_3}} |
| 中介适配器兼容性 | {{status_4}} | {{findings_4}} | {{next_steps_4}} |
| 广告预加载验证检查 | {{status_5}} | {{findings_5}} | {{next_steps_5}} |