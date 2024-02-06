---
title: Vue3+Node.js实现微信扫码登录流程
date: 2024-01-31 14:18:11
tags:
  - Vue3
  - Node.js
  - 微信API
categories:
  - [一些日常开发积累]
---

# 一、前言

最近刚刚实习，接到了个简单又不简单的需求：前端实现内嵌微信二维码扫码登录，而公司的技术栈用的是`Vue3+Pinia+TypeScript`。而我之前也仅仅只接触过微信小程序的微信授权登录，这种第三方 web 应用的微信授权登录还是头一次遇到。上网查了好多资料+问了 GPT 好久，最后整理了一下流程和大致的实现，希望对大家有所帮助~！

# 二、流程梳理

## 官方流程概述

1. **设置回调 URL**：在微信开放平台注册应用时，需要指定一个回调 URL，微信会在用户授权后使用这个 URL。
2. **用户授权**：用户扫描二维码并在微信中同意授权。
3. **微信重定向**：微信将用户重定向到设置的回调 URL，同时附带一个授权码。
4. **后端接口处理**：需要在这个回调 URL 上实现一个接口来接收授权码，并使用它向微信服务器请求 access_token 和用户信息。

## 开发者的任务

- **编写回调接口**：需要在自己的服务器上编写处理微信重定向的接口。这个接口应该能够处理微信的请求，提取授权码，并使用它来进行后续的操作。
- **安全性和稳定性**：确保这个接口能够安全、稳定地处理微信的请求，包括处理可能的错误情况。
- **获取用户信息**：利用获取到的 access_token 和 OpenID 向微信请求用户信息，并完成应用的登录或其他业务逻辑。

### 1. 前端（Vue 3）生成登录二维码

1. **用户访问登录页面**：用户打开 Vue 应用中的登录页面。
2. **请求后端获取二维码**：在页面上，通过 Vue 应用向 Express 后端发送请求以获取微信登录的二维码。
3. **后端处理请求**：Express 后端接收到请求后，调用微信的 API 来生成一个微信登录的 URL。
4. **生成二维码**：后端将微信登录的 URL 转换成二维码数据，并将这个二维码数据发送回 Vue 前端。
5. **显示二维码**：前端 Vue 应用接收到二维码数据后，展示给用户。

### 2. 用户扫码并授权

1. **用户扫描二维码**：用户使用微信扫描页面上显示的二维码。
2. **微信处理扫码**：微信处理用户的扫码请求，并要求用户确认登录授权。
3. **用户授权登录**：用户在微信中确认授权，允许应用获取其信息。

### 3. 后端（Express）处理授权回调

1. **微信重定向**：一旦用户授权，微信将用户重定向到在微信开放平台注册应用时指定的回调 URL。
2. **接收授权码**： Express 后端接收到包含授权码（code）的回调请求。
3. **请求访问令牌**：后端使用这个授权码向微信服务器请求访问令牌和用户的 OpenID。
4. **获取用户信息**：使用访问令牌，后端可以请求微信 API 获取用户的个人信息。

### 4. 完成用户登录

1. **创建用户会话**：根据获取到的用户信息，后端在应用中创建用户会话。
2. **发送响应到前端**：后端将登录成功的确认和必要的用户信息发送回前端。
3. **前端展示用户信息**：Vue 应用接收到后端的响应后，更新页面以显示用户已登录，展示用户的信息。

# 三、后端 API 接口相关

## 接口列举

如果要实现上述的登录流程，Express 后端需要编写如下的几个接口：

1. **获取登录二维码接口**：
   - **功能**：这个接口用于生成微信登录的 URL，并将其转换为二维码，供前端展示给用户。
   - **流程**：前端发起请求 -> 后端调用微信 API 获取授权 URL -> 后端生成二维码 -> 后端将二维码数据返回给前端。
2. **处理微信回调接口**：
   - **功能**：这个接口用于接收微信在用户授权后的回调。
   - **流程**：微信重定向用户到这个接口，携带授权码（code） -> 后端使用授权码请求微信服务器以获取访问令牌和用户信息 -> 后端处理并存储用户信息，创建用户会话。
3. **（可选）用户信息接口**：
   - **功能**：如果用户信息需要在不同的页面或组件中使用，可能需要一个专门的接口来获取当前登录用户的信息。
   - **流程**：前端发起请求 -> 后端验证用户会话 -> 后端返回当前登录用户的信息。
4. **（可选）登出接口**：
   - **功能**：用于处理用户登出操作。
   - **流程**：前端发起登出请求 -> 后端销毁用户会话 -> 后端返回登出成功的确认。

这些是实现微信扫码登录功能最基本的后端接口，根据具体需求可能还需要添加其他接口，比如用于用户注册、密码找回、用户权限管理等。

## 代码示例

```javascript
const express = require("express");
const axios = require("axios"); // 用于发起网络请求
const app = express();
const port = 3000;

// 假设已经有了微信的 AppID 和 AppSecret
const WECHAT_APPID = "AppID";
const WECHAT_SECRET = "AppSecret";
const WECHAT_REDIRECT_URI = "回调地址"; // 微信授权后的回调地址

// 获取微信登录二维码的接口
app.get("/api/wechat-login-qrcode", async (req, res) => {
  try {
    // 这里应该是调用微信 API 来获取授权 URL，并转换为二维码
    const wechatUrl = `https://open.weixin.qq.com/connect/qrconnect?appid=${WECHAT_APPID}&redirect_uri=${encodeURIComponent(
      WECHAT_REDIRECT_URI
    )}&response_type=code&scope=snsapi_login&state=STATE#wechat_redirect`;
    // 这里简化处理，直接返回 URL
    res.json({ url: wechatUrl });
  } catch (error) {
    res.status(500).send("服务器错误");
  }
});

// 微信回调接口，也就是用户扫码之后在手机上点击同意之后，需要进行重定向的目标URL
app.get("/wechat-callback", async (req, res) => {
  const code = req.query.code; // 获取微信回调返回的授权码
  try {
    // 使用授权码换取 access_token
    const tokenResponse = await axios.get(
      `https://api.weixin.qq.com/sns/oauth2/access_token?appid=${WECHAT_APPID}&secret=${WECHAT_SECRET}&code=${code}&grant_type=authorization_code`
    );
    const accessToken = tokenResponse.data.access_token;
    const openId = tokenResponse.data.openid;

    // 这里可以根据 accessToken 和 openId 获取用户信息，并进行登录处理

    res.send("登录成功");
  } catch (error) {
    res.status(500).send("授权失败");
  }
});

app.listen(port, () => {
  console.log(`服务器运行在 http://localhost:${port}`);
});
```

# 四、前端 Vue 组件二维码展示、接口调用相关

用户点击微信登录时，需要有个组件负责显示登录二维码，并处理用户点击事件来获取二维码。

```vue
<template>
  <div class="login-window">
    <button @click="getWechatQRCode">微信扫码登录</button>
    <div v-if="qrCodeUrl">
      <img :src="qrCodeUrl" alt="微信登录二维码" />
    </div>
  </div>
</template>

<script lang="ts">
import { ref, defineComponent } from "vue";
import axios from "axios";

export default defineComponent({
  setup() {
    const qrCodeUrl = ref<string | null>(null);

    const getWechatQRCode = async () => {
      try {
        // 请求后端接口获取二维码 URL
        const response = await axios.get("/api/wechat-login-qrcode");
        qrCodeUrl.value = response.data.url; // 这里假设后端返回的数据中有 url 字段
      } catch (error) {
        console.error("获取二维码失败:", error);
      }
    };

    return {
      qrCodeUrl,
      getWechatQRCode,
    };
  },
});
</script>

<style scoped>
.login-window {
  text-align: center;
  padding: 20px;
}

.login-window img {
  margin-top: 20px;
  max-width: 200px;
  border: 1px solid #ddd;
}
</style>
```

# 五、比较迷惑的几个问题

## 微信重定向到底是重定向到哪里？

当在微信开放平台创建应用时，会被要求提供一个“回调 URL”（也称为重定向 URI）。这个 URL**应该指向服务器上的一个接口**，在上述的应用当中应该指的是`/wechat-callback`这个接口。在用户通过微信完成扫码并授权之后，微信会自动将用户的浏览器重定向到这个 URL。

这个重定向动作的主要目的是将用户带回应用或网站，并在这个过程中，微信会向提供一些关键信息，主要是“授权码”（code）。

## 授权码（Code）起到什么作用？

授权码是 OAuth 2.0 授权流程的一个重要组成部分。它是一种短暂有效的码，用来在用户授权之后安全地在微信和服务器之间传递信息。

1. **用户授权后获取授权码**：当用户在微信中扫码并同意授权后，微信会生成这个授权码，并将它附加在重定向请求中发送给服务器。
2. **服务器使用授权码获取 access_token**：服务器接收到授权码后，需要立即使用它来向微信请求 access_token。这个过程通常是通过在服务器后端发起另一个 HTTP 请求来完成的。
3. **用 access_token 获取用户信息**：一旦服务器获取了 access_token，就可以使用它来访问微信的 API，获取用户的信息，例如用户的 OpenID、昵称、头像等。

## 实际的登陆操作流程是什么？

- **设置回调 URL**：在微信开放平台的应用设置中，需要设定一个回调 URL，比如 `https://yourdomain.com/wechat-callback`。
- **用户扫码授权后的流程**：
  1. 用户在网站上点击“微信登录”，扫描二维码。
  2. 用户在微信中确认授权。
  3. 微信将用户的浏览器重定向到提供的回调 URL，并在 URL 中附加授权码，如 `https://yourdomain.com/wechat-callback?code=AUTHORIZATION_CODE`。
  4. 服务器在接收到带有授权码的请求后，使用这个授权码向微信请求 access_token。
  5. 微信响应 access_token 请求，并发送 access_token 给服务器。
  6. 服务器使用这个 access_token 来获取用户信息，并完成登录或其他业务逻辑。

## 前端 Vue3 应用怎么知道用户已经在微信平台授权登录？

### 1. 轮询（Polling）

在这种方法中，前端 Vue 应用在用户开始扫描二维码后，会定期向后端发送请求以查询登录状态。

流程：

1. 用户扫描二维码并在微信中授权。
2. 前端开始定期发送请求到后端，询问用户是否已经成功登录。
3. 一旦用户在微信中授权，后端处理微信的回调，并更新用户的登录状态。
4. 在下一次轮询时，前端得知用户已登录，并进行相应的 UI 更新或页面跳转。

### 2. WebSocket

如果应用已经使用了 WebSocket，这是一种更现代且实时的方法。WebSocket 允许服务器主动向前端推送信息。

流程：

1. 用户打开登录页面，前端建立 WebSocket 连接。
2. 用户扫描二维码并在微信中授权。
3. 后端在处理微信回调后，通过 WebSocket 向前端发送用户已登录的消息。
4. 前端接收到消息后更新 UI 或执行其他逻辑。

### 3. 重定向和前端路由

在这种方法中，微信授权后的重定向不仅仅是到后端接口，而是重定向回整个前端应用的特定路由。

流程：

1. 用户扫描二维码并在微信中授权。
2. 微信将用户重定向回前端应用的特定路由，比如 `https://yourapp.com/login/callback`。
3. 当用户到达这个路由时，前端发起请求到后端以确认登录状态。
4. 后端确认用户已登录，并向前端返回确认消息。
5. 前端根据这个消息更新 UI 或跳转到用户的主页。
