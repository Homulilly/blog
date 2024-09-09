---
title: 在 OpenAI Translator 中使用 Gemini Pro
date: 2024-02-23 14:24:26
tags:
 - Gemini
categories:
 - Note
---

OpenAI Translator 是很好用的翻译工具，但是使用 OpenAI 的 Key 是需要付费的，~~而且想付费也需要克服一些困难~~。  
这里感谢财大气粗的 Google 提供了免费的 Gemini Pro。   

首先要去 [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey) 创建 API Key。

然后需要去 [Google Cloud Console](https://console.cloud.google.com/vertex-ai) 开启 `Vertex AI API`，Google 刚推出时我尝试使用时在 OpenAI Translator 中一直报错，开启了 `Vertex AI API` 就可以使用了。

最后将申请的 API Key 填入 OpenAI Translator 中就可以了。 

![OpenAI Translator](https://m.nep.me/blog/post/openai-translator-gemini.png) 

如果使用魔法，需要将下面的域名转发至允许使用的地区，香港地区是无法使用的。
```
- DOMAIN-SUFFIX,generativelanguage.googleapis.com,AI
```

API 的使用情况可以在 [Google Cloud Dashboard](https://console.cloud.google.com/apis/dashboard) 中查看

{% note success %}
2024.4 月初收到邮件提示，Google 说将在 5 月 2 号起对 Cloud Billing 账号关联的项目的 Gemini API 收费。      
>注意：本通知仅针对 Google AI for Developers 中的 Gemini API（在 Cloud 控制台中称为 Generative Language API），而与 Vertex AI Gemini API 无关。  
  
访问了项目相关的 `https://console.cloud.google.com/apis/library/generativelanguage.googleapis.com?project=<ProjectName>` 看到功能是未启用状态。  
应该是没有影响。  
不过保险起见，还是可以前往 [Google Cloud - 我的结算 - 我的项目](https://console.cloud.google.com/billing/projects) 停用结算功能。
{% endnote %}