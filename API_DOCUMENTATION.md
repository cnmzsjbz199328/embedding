# Embedding Service API Documentation

## 1. 简介

本服务提供将文本转换为高质量向量（Embedding）的能力。它基于强大的 `sentence-transformers/all-MiniLM-L6-v2` 模型，能够将任意文本片段编码为一个包含 384 个维度的浮点数数组。这些向量可以用于各种下游任务，如语义搜索、文本聚类、相似度计算等。

**服务基础 URL**: `https://embedding.badtom.dpdns.org`

## 2. 认证

当前部署的服务是开放的，**没有设置认证机制**。任何知道此 URL 的人都可以访问。

在生产环境中，强烈建议通过 Cloudflare Access 或在应用层面增加 API 密钥等方式来保护此端点。

---

## 3. API 端点

### 3.1 健康检查

检查服务的运行状态和所加载模型的信息。

- **URL**: `/`
- **方法**: `GET`
- **描述**: 这是服务的基础端点，用于简单的健康检查。
- **请求示例**: 无需参数或请求体。

- **成功响应 (200 OK)**:
  ```json
  {
    "message": "All-MiniLM-L6-v2 Embedding Service",
    "status": "healthy",
    "model": "sentence-transformers/all-MiniLM-L6-v2",
    "dimensions": 384
  }
  ```
- **字段说明**:
  - `message`: 服务信息。
  - `status`: `healthy` 表示服务正常。
  - `model`: 当前加载的 Transformer 模型名称。
  - `dimensions`: 模型输出的向量维度。

### 3.2 创建 Embedding

这是服务的核心功能，用于将输入的文本向量化。

- **URL**: `/embed`
- **方法**: `POST`
- **描述**: 发送一段文本，获取其对应的 384 维向量。

- **请求体 (Request Body)**:
  - **Content-Type**: `application/json`
  - **格式**: 一个 JSON 对象，包含一个名为 `text` 的键。
  - **示例**:
    ```json
    {
      "text": "您想要进行向量化的任何文本内容。"
    }
    ```

- **成功响应 (200 OK)**:
  - **格式**: 一个 JSON 对象，包含向量、维度和处理时间。
  - **示例** (为简洁起见，向量数组已被缩短):
    ```json
    {
      "embedding": [
        -0.013980641029775143,
        -0.06696616858243942,
        0.01995718665421009,
        // ... 更多浮点数 ...
        0.007487795781344175
      ],
      "dimensions": 384,
      "process_time": 3.554
    }
    ```
  - **字段说明**:
    - `embedding`: 包含 384 个浮点数的数组，即文本的向量表示。
    - `dimensions`: 向量维度，应为 384。
    - `process_time`: 服务器处理该请求所花费的时间（单位：秒）。

- **错误响应**:
  - `422 Unprocessable Entity`: 如果请求体不是有效的 JSON 或缺少 `text` 字段，服务器将返回此错误。

---

## 4. 使用示例

### 4.1 使用 cURL

这是一个简单的命令行示例，用于测试服务。

```bash
curl -X POST https://embedding.badtom.dpdns.org/embed \
     -H "Content-Type: application/json" \
     -d '{"text": "Hello world, this is a test from curl."}'
```

### 4.2 使用 Python (requests 库)

在 Python 项目中调用此 API 的推荐方法。

```python
import requests
import json

api_url = "https://embedding.badtom.dpdns.org/embed"
headers = {"Content-Type": "application/json"}

def get_embedding(text_to_embed):
    """获取给定文本的向量。"""
    data = {"text": text_to_embed}
    try:
        response = requests.post(api_url, headers=headers, json=data)
        response.raise_for_status()  # 如果状态码不是 2xx，则引发异常
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"调用 API 时出错: {e}")
        return None

# --- 使用示例 ---
my_text = "这是一个用 Python 进行的测试。"
embedding_data = get_embedding(my_text)

if embedding_data:
    print(f"成功获取向量，维度为: {embedding_data['dimensions']}")
    # 打印向量的前5个维度
    print(f"向量前5维: {embedding_data['embedding'][:5]}")

```

### 4.3 使用 JavaScript (Fetch API)

在前端或 Node.js 环境中调用此 API 的方法。

```javascript
async function getEmbedding(text) {
  const apiUrl = 'https://embedding.badtom.dpdns.org/embed';
  
  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ text: text }),
    });

    if (!response.ok) {
      throw new Error(`HTTP 错误! 状态码: ${response.status}`);
    }

    const data = await response.json();
    return data;

  } catch (error) {
    console.error('调用 API 时出错:', error);
    return null;
  }
}

// --- 使用示例 ---
const myText = 'This is a test from JavaScript.';

getEmbedding(myText).then(data => {
  if (data) {
    console.log(`成功获取向量，维度为: ${data.dimensions}`);
    // 打印向量的前5个维度
    console.log('向量前5维:', data.embedding.slice(0, 5));
  }
});
```