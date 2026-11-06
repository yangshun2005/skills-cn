# 合作伙伴链接

合作伙伴链接将广告主账号连接到数据合作伙伴账号。数据合作伙伴随后使用其**自有**数据合作伙伴账号的凭据，管理广告主账号中的受众群体或事件。完整指南请参阅[合作伙伴链接](https://developers.google.com/data-manager/api/devguides/accounts/partner-links.md.txt)。

## 数据合作伙伴工作流

广告主也可以在 Google Ads、Display & Video 360 或 Google Ad Manager 界面中管理合作伙伴链接。当这些界面不可用，或需要在合作伙伴自己的应用中提供链接功能时，请使用 `PartnerLinkService`。

1.  **获取短期广告主凭据**，该凭据属于广告主账号中的某个 Google 账号用户，且仅请求以下范围：

    `https://www.googleapis.com/auth/datamanager.partnerlink`

    在合作伙伴的应用中构建 OAuth 流程。使用 **Web 服务器**或 **Web 浏览器** OAuth 客户端（而非 Desktop），并遵循
    [适用于 Web 服务器应用的 OAuth 2.0](https://developers.google.com/identity/protocols/oauth2/web-server)
    或
    [适用于客户端 Web 应用的 OAuth 2.0](https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow)。
2.  **使用广告主凭据创建合作伙伴链接**。请求字段和示例请参阅
    [创建或删除合作伙伴链接](https://developers.google.com/data-manager/api/devguides/accounts/partner-links/create-partner-link.md.txt)。
3.  **丢弃广告主凭据。** 不再需要它们。
4.  **对于之后的所有请求**，使用有权访问**数据合作伙伴账号**的 Google 账号或服务账号的凭据。

## OAuth 应用验证

`datamanager.partnerlink` 范围属于敏感范围，因此合作伙伴的应用必须先完成 OAuth 应用验证，广告主才能授权。否则，广告主会看到未验证应用界面。

验证流程的详细信息请参阅
[OAuth 应用验证](https://developers.google.com/data-manager/api/devguides/concepts/oauth-verification.md.txt)。