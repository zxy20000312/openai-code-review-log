[{"title":"删除标准 isFinal() getter 可能破坏 JavaBeans/JSON 兼容性","description":"AgentAction 原来提供 boolean isFinal() 作为标准 getter，现仅保留 getIsFinal()。如果该类被 Jackson、Fastjson 等框架序列化或反序列化，字段名可能从 ‘final’ 变为 ‘isFinal’，与 LLM 交互的 JSON 结构不兼容；同时也不符合 JavaBeans 命名规范，建议保留 isFinal() 或显式添加 @JsonProperty 注解。","filePath":"openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/model/AgentAction.java","severity":"MEDIUM"},{"title":"环境变量重命名导致已有配置静默失效","description":"OpenAiCodeReviewService 将 CHATGLM_EMBEDDING_APIHOST/CHATGLM_APIKEYSECRET 改为 EMBEDDING_APIHOST/LLM_APIKEYSECRET。若部署环境仍使用旧变量，apiKey 会变为空，代码会跳过 embedding 初始化，导致 RAG 功能静默关闭且无告警。建议兼容旧变量名或提供迁移说明。","filePath":"openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java","severity":"MEDIUM"}]

---
## 二次验证结果

验证统计: 确认=2, 误报=0, 不确定=0

- [CONFIRMED] MEDIUM - 删除标准 isFinal() getter 可能破坏 JavaBeans/JSON 兼容性 (openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/model/AgentAction.java)
- [CONFIRMED] MEDIUM - 环境变量重命名导致已有配置静默失效 (openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java)
