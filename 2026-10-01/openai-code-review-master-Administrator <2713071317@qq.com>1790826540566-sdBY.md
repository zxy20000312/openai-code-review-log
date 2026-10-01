[{"title":"EmbeddingService 请求体 input 格式与 OpenAI 兼容接口不匹配","description":"默认 host 已切换到阿里云 DashScope 的 compatible_mode/v1/embeddings 接口，但代码将 input 从字符串改为 Map.of(text, text) 对象。该接口要求 input 为字符串或数组，传入对象会导致 embedding 请求失败。","filePath":"openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/rag/EmbeddingService.java","severity":"HIGH"},{"title":"AgentAction 同时存在 isFinal 和 getIsFinal 两个 getter","description":"boolean 字段 isFinal 同时暴露 isFinal() 和 getIsFinal()，Jackson/Fastjson 可能将其识别为 final 与 isFinal 两个属性，造成 JSON 字段重复、序列化歧义或反序列化失败。","filePath":"openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/model/AgentAction.java","severity":"MEDIUM"},{"title":"默认 embedding host 与环境变量命名不匹配","description":"默认 host 改为阿里云 DashScope，但读取的 key 仍来自 CHATGLM_APIKEYSECRET 且变量名仍为 CHATGLM_EMBEDDING_APIHOST。未显式配置 host 时，使用 ChatGLM key 请求阿里云会鉴权失败，且变量命名存在误导。","filePath":"openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java","severity":"MEDIUM"}]

---
## 二次验证结果

验证统计: 确认=3, 误报=0, 不确定=0

- [CONFIRMED] HIGH - EmbeddingService 请求体 input 格式与 OpenAI 兼容接口不匹配 (openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/rag/EmbeddingService.java)
- [CONFIRMED] MEDIUM - AgentAction 同时存在 isFinal 和 getIsFinal 两个 getter (openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/model/AgentAction.java)
- [CONFIRMED] MEDIUM - 默认 embedding host 与环境变量命名不匹配 (openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java)
