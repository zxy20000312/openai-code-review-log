# 代码评审报告

## 评审发现

| # | 严重程度 | 问题 | 文件 | 状态 |
|---|----------|------|------|------|
| 1 | HIGH | 测试代码中仍存在硬编码 API 密钥 | openai-code-review-sdk/src/test/java/plus/gaga/middleware/sdk/test/ApiTest.java | CONFIRMED |
| 2 | HIGH | LLM+RAG Benchmark 实际未调用 code_search 工具 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/agent/BenchmarkRunner.java | UNCERTAIN |
| 3 | MEDIUM | Benchmark 中 missedIssues 计算可能为负数 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/agent/BenchmarkRunner.java | CONFIRMED |
| 4 | LOW | BenchmarkRunner 对空 cases 列表除零 | openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/agent/BenchmarkRunner.java | CONFIRMED |
| 5 | LOW | 工作流使用过时的 setup-java@v2 和 adopt 发行版 | .github/workflows/benchmark.yml | CONFIRMED |

## 问题详情

### 1. [CONFIRMED] 测试代码中仍存在硬编码 API 密钥

- **严重程度**: HIGH
- **文件**: openai-code-review-sdk/src/test/java/plus/gaga/middleware/sdk/test/ApiTest.java
- **验证状态**: CONFIRMED

ApiTest.main 中仍保留硬编码的智谱 API key secret，并通过 BearerTokenUtils 生成 token，存在密钥泄露风险，应改为从环境变量或 Secret 读取。

### 2. [UNCERTAIN] LLM+RAG Benchmark 实际未调用 code_search 工具

- **严重程度**: HIGH
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/agent/BenchmarkRunner.java
- **验证状态**: UNCERTAIN

BenchmarkRunner.gatherRAGContext 查找名为 code_search 的工具，但 ApiTest 中传入的 allTools 只有 GetDiffTool(get_diff) 和 ReadFileTool(read_file)，导致该方法始终返回空字符串，LLM+RAG 与 LLM 配置实际相同，Benchmark 对比结果无效。

### 3. [CONFIRMED] Benchmark 中 missedIssues 计算可能为负数

- **严重程度**: MEDIUM
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/agent/BenchmarkRunner.java
- **验证状态**: CONFIRMED

missedIssues = expectedIssues.size() - truePositives，而 truePositives 统计的是命中期望关键词的 reported issues 数量，不是唯一命中期望项数。若模型返回多个同类问题，tp 可能大于期望关键词数量，导致 missedIssues 为负，指标失真。

### 4. [CONFIRMED] BenchmarkRunner 对空 cases 列表除零

- **严重程度**: LOW
- **文件**: openai-code-review-sdk/src/main/java/plus/gaga/middleware/sdk/domain/service/agent/BenchmarkRunner.java
- **验证状态**: CONFIRMED

run 方法在计算平均延迟和平均 token 时使用 cases.size() 作为除数，未校验 cases 是否为空；传入空列表时会抛出 ArithmeticException。

### 5. [CONFIRMED] 工作流使用过时的 setup-java@v2 和 adopt 发行版

- **严重程度**: LOW
- **文件**: .github/workflows/benchmark.yml
- **验证状态**: CONFIRMED

.github/workflows/benchmark.yml 使用 actions/setup-java@v2 且 distribution 为 adopt，建议升级到 setup-java@v4 和 temurin/adoptium，避免旧版本弃用导致工作流不稳定。

---

## 二次验证统计

| 指标 | 数值 |
|------|------|
| 确认(Confirmed) | 4 |
| 误报(False Positive) | 0 |
| 不确定(Uncertain) | 1 |
