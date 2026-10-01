# 代码评审报告

## 评审发现

| # | 严重程度 | 问题 | 文件 | 状态 |
|---|----------|------|------|------|
| 1 | HIGH | RetryableLLM 超时线程泄漏及内存可见性问题 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/openai/impl/RetryableLLM.java | CONFIRMED |
| 2 | HIGH | GitHubApi.getPullRequestDiff 未检查 HTTP 响应码 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/git/GitHubApi.java | CONFIRMED |
| 3 | MEDIUM | OpenAiCodeReviewService 重复解析环境变量且未校验数组越界 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java | CONFIRMED |
| 4 | MEDIUM | McpServer.handleRequest 错误响应丢失请求 id | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpServer.java | CONFIRMED |
| 5 | MEDIUM | McpClient 资源泄漏及线程安全问题 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpClient.java | CONFIRMED |
| 6 | LOW | GitCommand.author 未判空且文件名字符过滤不完整 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/git/GitCommand.java | CONFIRMED |
| 7 | LOW | McpServer.handleToolsList 返回的 inputSchema 不完整 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpServer.java | CONFIRMED |

## 问题详情

### 1. [CONFIRMED] RetryableLLM 超时线程泄漏及内存可见性问题

- **严重程度**: HIGH
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/openai/impl/RetryableLLM.java
- **验证状态**: CONFIRMED

callWithTimeout 每次调用 new Thread 并在 join(timeoutMs) 之后仅调用 thread.interrupt() 而不强制停止，线程可能继续运行且结果数组未同步，存在线程泄漏和内存可见性风险。建议使用 ExecutorService 或 CompletableFuture 实现超时，并设置守护线程。

### 2. [CONFIRMED] GitHubApi.getPullRequestDiff 未检查 HTTP 响应码

- **严重程度**: HIGH
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/git/GitHubApi.java
- **验证状态**: CONFIRMED

该方法直接使用 conn.getInputStream()，对于 4xx/5xx 响应会抛出异常且无法获取错误体，且未复用 readResponse 统一错误处理。建议统一复用 readResponse，避免错误信息丢失。

### 3. [CONFIRMED] OpenAiCodeReviewService 重复解析环境变量且未校验数组越界

- **严重程度**: MEDIUM
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java
- **验证状态**: CONFIRMED

代码中两次重复从环境变量读取并解析 PR 信息，且 githubRepo.split("/") 未校验长度，可能导致 ArrayIndexOutOfBoundsException。同时重复创建 GitHubApi 实例，逻辑冗余。建议提取方法并添加校验。

### 4. [CONFIRMED] McpServer.handleRequest 错误响应丢失请求 id

- **严重程度**: MEDIUM
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpServer.java
- **验证状态**: CONFIRMED

在 catch 块中返回 McpMessage.error(0, -32603, ...)，id 硬编码为 0，客户端无法将错误关联到对应请求。建议从解析后的 request 中获取 id，若解析失败则返回 id 为 null。

### 5. [CONFIRMED] McpClient 资源泄漏及线程安全问题

- **严重程度**: MEDIUM
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpClient.java
- **验证状态**: CONFIRMED

connect() 启动进程后未在异常时关闭相关资源；requestId 递增非原子操作，多线程调用时可能冲突；sendRequest 未考虑 JSON-RPC 被拆分为多行的情况，可能不兼容标准 MCP 内容长度前缀。

### 6. [CONFIRMED] GitCommand.author 未判空且文件名字符过滤不完整

- **严重程度**: LOW
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/git/GitCommand.java
- **验证状态**: CONFIRMED

safeAuthor = author.replaceAll(...) 若 author 为 null 会抛出 NPE。同时只替换了部分非法字符，未覆盖 Windows 下其他非法字符如 ! 等。建议增加 null 判断并完善替换规则。

### 7. [CONFIRMED] McpServer.handleToolsList 返回的 inputSchema 不完整

- **严重程度**: LOW
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpServer.java
- **验证状态**: CONFIRMED

工具列表返回的 inputSchema 只包含 type 和空 properties，不符合 MCP 工具描述规范，可能影响客户端解析。建议为每个工具提供完整参数 schema。

---

## 二次验证统计

| 指标 | 数值 |
|------|------|
| 确认(Confirmed) | 7 |
| 误报(False Positive) | 0 |
| 不确定(Uncertain) | 0 |
