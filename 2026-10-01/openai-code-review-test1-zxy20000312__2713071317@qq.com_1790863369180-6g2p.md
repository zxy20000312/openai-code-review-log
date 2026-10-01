# 代码评审报告

## 评审发现

| # | 严重程度 | 问题 | 文件 | 状态 |
|---|----------|------|------|------|
| 1 | HIGH | RetryableLLM 超时线程泄漏且 decide 未超时 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/openai/impl/RetryableLLM.java | CONFIRMED |
| 2 | HIGH | GitHubApi 缺少超时设置且错误处理不一致 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/git/GitHubApi.java | CONFIRMED |
| 3 | MEDIUM | PR 处理代码重复且缺乏结果校验 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java | CONFIRMED |
| 4 | MEDIUM | CI 工作流使用过时版本且构建过重 | .github/workflows/pr-review.yml | CONFIRMED |
| 5 | LOW | McpClient 未读取 stderr 可能导致阻塞 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpClient.java | CONFIRMED |
| 6 | LOW | McpServer 缺少 params 空值校验 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpServer.java | CONFIRMED |

## 问题详情

### 1. [CONFIRMED] RetryableLLM 超时线程泄漏且 decide 未超时

- **严重程度**: HIGH
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/openai/impl/RetryableLLM.java
- **验证状态**: CONFIRMED

使用普通 Thread 执行 LLM 请求，超时后仅调用 interrupt，若底层客户端不响应中断，线程不会停止，可能造成线程泄漏；且 decide 方法未复用超时控制，如果 delegate 内部阻塞，将永久挂起整个评审流程。

### 2. [CONFIRMED] GitHubApi 缺少超时设置且错误处理不一致

- **严重程度**: HIGH
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/git/GitHubApi.java
- **验证状态**: CONFIRMED

HttpURLConnection 未设置 connectTimeout 和 readTimeout，可能导致请求无限挂起；getPullRequestDiff 直接使用 getInputStream() 未检查响应码，其他方法通过 readResponse 统一处理，不一致易导致异常信息不明确。

### 3. [CONFIRMED] PR 处理代码重复且缺乏结果校验

- **严重程度**: MEDIUM
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/impl/OpenAiCodeReviewService.java
- **验证状态**: CONFIRMED

OpenAiCodeReviewService 中添加 PR 工具和发布评论时重复解析 owner/repo、创建 GitHubApi 实例，代码冗余可维护性差；发布评论前未检查 result 或 result.getAnswer() 是否为 null，可能引发 NPE；仓库名 split 逻辑假设固定格式。

### 4. [CONFIRMED] CI 工作流使用过时版本且构建过重

- **严重程度**: MEDIUM
- **文件**: .github/workflows/pr-review.yml
- **验证状态**: CONFIRMED

workflow 使用 actions/checkout@v2 和 actions/setup-java@v2 过时版本；在 PR 评审阶段执行 mvn clean install 会构建整个项目，耗时长且可能因构建失败阻断评审流程，建议改用更轻量的依赖获取方式。

### 5. [CONFIRMED] McpClient 未读取 stderr 可能导致阻塞

- **严重程度**: LOW
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpClient.java
- **验证状态**: CONFIRMED

ProcessBuilder.redirectErrorStream(false) 但未启动单独线程读取 stderr，若 MCP 服务器输出较多错误日志会填满缓冲区，导致进程阻塞，影响工具调用。

### 6. [CONFIRMED] McpServer 缺少 params 空值校验

- **严重程度**: LOW
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/infrastructure/mcp/McpServer.java
- **验证状态**: CONFIRMED

handleToolsCall 中直接使用 params.getString("name")，若客户端发送 tools/call 请求缺少 params 对象会抛出 NPE，且异常响应使用 id=0 导致客户端无法关联请求。

---

## 二次验证统计

| 指标 | 数值 |
|------|------|
| 确认(Confirmed) | 6 |
| 误报(False Positive) | 0 |
| 不确定(Uncertain) | 0 |
