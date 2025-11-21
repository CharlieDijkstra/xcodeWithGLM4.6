# Xcode GLM-4.6 Integration Tutorial

This tutorial will guide you through integrating GLM-4.6 with Xcode for enhanced code completion and AI assistance.

## Prerequisites

- Xcode version with Coding Intelligence support
- GLM Coding Plan API key from Bigmodel

## Configuration Steps

### 1. Open Xcode Model Provider Settings

In Xcode, add GLM as a model provider:
- Click on **Xcode** in the menu bar
- Select **Settings...**
- Navigate to the **Intelligence** tab
- Click the **Add a Model Provider** button

![Xcode Add a Model Provider](xcode.png)

### 2. Configure GLM API

Fill in the following fields with the provided configuration:

| Field | Value |
|-------|-------|
| **URL** | `https://open.bigmodel.cn/api/anthropic` |
| **API Key** | `你的Key` (replace with your actual API key) |
| **API Key Header** | `Authorization` |
| **Description** | `Bigmodel` |

### 3. URL Forwarding Script

Since GLM uses a different endpoint structure than the standard Anthropic API, you'll need to use a URL forwarding script. This script will redirect Xcode's Anthropic API calls to the correct GLM endpoint.

Create the following forwarding function:

```javascript
async function onRequest(context, url, request) {
  if (url.includes('/api/anthropic/v1/chat/completions')) {
    // Build new URL
    const newUrl = 'https://open.bigmodel.cn/api/coding/paas/v4/chat/completions';
    console.log('Replace URL:', url, '->', newUrl);

    // Update request URL and related properties
    request.url = newUrl;
    request.host = 'open.bigmodel.cn';
    request.path = '/api/coding/paas/v4/chat/completions';
  }
  return request;
}
```

---

*This configuration allows Xcode to leverage GLM-4.6's powerful code completion capabilities while maintaining compatibility with Xcode's Anthropic API interface.*