# Xcode GLM-4.6 集成教程

本教程将指导您如何在 Xcode 中集成 GLM-4.6，以获得增强的代码补全和 AI 辅助功能。

## 前置要求

- 支持 Coding Intelligence 功能的 Xcode 版本
- 智谱 GLM Coding Plan 的 API Key

## 配置步骤

### 1. 打开 Xcode 模型提供商设置

在 Xcode 中添加 GLM 模型提供商：
- 点击菜单栏中的 **Xcode**
- 选择 **Settings...**（设置...）
- 导航到 **Intelligence** 选项卡
- 点击 **Add a Model Provider** 按钮

![Xcode Add a Model Provider](xcode.png)

### 2. 配置 GLM API

使用以下配置信息填写相应字段：

| 字段 | 值 |
|-------|-------|
| **URL** | `https://open.bigmodel.cn/api/anthropic` |
| **API Key** | `你的Key`（替换为你的实际 API Key） |
| **API Key Header** | `Authorization` |
| **Description** | `Bigmodel` |

### 3. API 转发配置

由于 GLM 使用与标准 Anthropic API 不同的端点结构，你需要配置 API 请求转发，将 Xcode 的 Anthropic API 调用重定向到正确的 GLM 端点。

#### 3.1 使用 Loon 代理软件转发

1. **配置复写规则**

   在 Loon 配置中添加以下复写规则：
   ```
   [Rewrite]
   # 将 Anthropic API 路径替换为 GLM Coding API 路径
   ^https://open.bigmodel.cn/api/anthropic/v1/chat/completions - reject
   ^https://open.bigmodel.cn/api/coding/paas/v4/chat/completions - reject
   ^https://open.bigmodel.cn/api/anthropic/v1/chat/completions https://open.bigmodel.cn/api/coding/paas/v4/chat/completions 302
   ```

2. **配置 HTTPS 解密（MITM）**

   在 Loon 设置中：
   - 开启 HTTPS 解密（MITM）
   - 添加域名到 MITM 主机名：`open.bigmodel.cn`

3. **使用脚本转发（推荐）**

   在 Loon 脚本模块中添加以下脚本：
   ```javascript
   async function onRequest(context, url, request) {
     if (url.includes('/api/anthropic/v1/chat/completions')) {
       const newUrl = 'https://open.bigmodel.cn/api/coding/paas/v4/chat/completions';
       console.log('Loon 转发 URL:', url, '->', newUrl);

       request.url = newUrl;
       request.host = 'open.bigmodel.cn';
       request.path = '/api/coding/paas/v4/chat/completions';
     }
     return request;
   }
   ```

#### 3.2 使用 Proxyman 转发

1. **创建映射规则**

   在 Proxyman 中：
   - 打开 Mapping 功能
   - 添加新的映射规则
   - 匹配 URL：`https://open.bigmodel.cn/api/anthropic/v1/chat/completions`
   - 转发到：`https://open.bigmodel.cn/api/coding/paas/v4/chat/completions`

2. **使用脚本功能**

   在 Proxyman 的 Script 功能中添加以下脚本：
   ```javascript
   // 匹配请求
   if (url.includes('/api/anthropic/v1/chat/completions')) {
     // 构建新的 URL
     const newUrl = 'https://open.bigmodel.cn/api/coding/paas/v4/chat/completions';
     console.log('Proxyman 转发 URL:', url, '->', newUrl);

     // 更新请求
     request.url = newUrl;
     request.host = 'open.bigmodel.cn';
     request.path = '/api/coding/paas/v4/chat/completions';
   }
   ```

3. **配置 SSL 证书**

   - 安装 Proxyman 的根证书
   - 信任证书（iOS/Android/macOS）
   - 确保 `open.bigmodel.cn` 在 SSL 信任列表中

#### 3.3 其他代理软件

其他代理软件（如 Surge、Clash、Quantumult X 等）的配置原理类似：

1. **核心要求**：
   - 开启 HTTPS 解密（MITM）
   - 添加域名 `open.bigmodel.cn` 到解密列表
   - 配置 URL 映射或重写规则

2. **基本转发逻辑**：
   ```
   原请求：https://open.bigmodel.cn/api/anthropic/v1/chat/completions
   转发到：https://open.bigmodel.cn/api/coding/paas/v4/chat/completions
   ```

3. **通用脚本模板**：
   ```javascript
   async function handleRequest(context, url, request) {
     if (url.includes('/api/anthropic/v1/chat/completions')) {
       request.url = 'https://open.bigmodel.cn/api/coding/paas/v4/chat/completions';
       request.host = 'open.bigmodel.cn';
       request.path = '/api/coding/paas/v4/chat/completions';
     }
     return request;
   }
   ```

---

