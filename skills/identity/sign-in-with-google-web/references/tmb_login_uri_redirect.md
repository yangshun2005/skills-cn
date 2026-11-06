# 模式 B：令牌中介后端（`login_uri` 表单 POST 重定向）方案

本参考文档提供 **模式 B（通过 `login_uri` 重定向的令牌中介后端 — IETF `draft-ietf-oauth-browser-based-apps` § 6.2）** 的完整实现方案，适用于服务端渲染的 Web 应用和直接表单 POST 重定向。

--------------------------------------------------------------------------------

## 1. 前端配置（直接服务端表单 POST）

```html
<!-- HTML 数据属性配置 -->
<div id="g_id_onload"
     data-client_id="YOUR_CLIENT_ID.apps.googleusercontent.com"
     data-login_uri="https://example.com/api/auth/callback"
     data-auto_select="true"
     data-ux_mode="redirect">
</div>
<div class="g_id_signin" data-type="standard" data-ux_mode="redirect"></div>

<!-- 或以编程方式初始化 GIS -->
<script>
  google.accounts.id.initialize({
    client_id: 'YOUR_CLIENT_ID.apps.googleusercontent.com',
    login_uri: 'https://example.com/api/auth/callback',
    ux_mode: 'redirect',
    auto_select: true
  });
  google.accounts.id.renderButton(document.getElementById('signin-btn'), {
    theme: 'outline',
    size: 'large'
  });
</script>
```

--------------------------------------------------------------------------------

## 2. 后端表单回调处理器（FastAPI / Python）

```python
from fastapi import Cookie, FastAPI, Form, HTTPException
from fastapi.responses import RedirectResponse
from google.auth.transport import requests
from google.oauth2 import id_token

app = FastAPI()
GOOGLE_CLIENT_ID = "YOUR_CLIENT_ID.apps.googleusercontent.com"


@app.post("/api/auth/callback")
async def google_login_redirect(
    credential: str = Form(...),
    g_csrf_token: str | None = Form(None),
    g_csrf_token_cookie: str | None = Cookie(None, alias="g_csrf_token"),
):
  # 1. 使用 login_uri 重定向时进行双提交 CSRF 令牌校验
  if (
      not g_csrf_token
      or not g_csrf_token_cookie
      or g_csrf_token != g_csrf_token_cookie
  ):
    raise HTTPException(
        status_code=400, detail="CSRF check failed on login_uri redirect"
    )

  try:
    # 2. 以密码学方式验证 ID 令牌
    id_info = id_token.verify_oauth2_token(
        credential, requests.Request(), GOOGLE_CLIENT_ID
    )
    user_id = id_info["sub"]
    user_email = id_info["email"]

    # 3. 建立会话并签发安全 Cookie 重定向
    response = RedirectResponse(url="/dashboard", status_code=303)
    response.set_cookie(
        key="session_id",
        value=create_server_session(user_id, user_email),
        httponly=True,
        secure=True,
        samesite="lax",
    )
    return response
  except ValueError as e:
    raise HTTPException(
        status_code=401, detail="Invalid credential token"
    ) from e
```