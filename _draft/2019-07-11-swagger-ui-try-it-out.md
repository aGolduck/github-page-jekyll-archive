---
title: Bypass quirk of Swagger-UI try-it-out
tags: [swagger, openapi]
---

Swagger-UI 有一个很有用的特性，就是可以用 `Try it out` 按钮直接试用，配置好授权策略以后，还可以直接使用登陆态。但是我自己在使用的过程中经常会遇到一个问题。经常会以下的错误。

![firefox error response]({{ site.url }}/assets/firefox-error-response.jpg)

使用 Chrome 浏览器会返回 `TypeError: Failed to fetch`。打开 `Network` 面板红红的一条记录，点开 Response: `Failed to load response data`
