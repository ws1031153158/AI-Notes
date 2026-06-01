# Tool Calling 超时
设置超时熔断（如5s），未返回自动降级为缓存响应，避免对话卡死
# ReAct 死循环检测
通过 Hash 比对 + 迭代阈值双重校验，实时拦截 Tool Calling 死循环
# API 指数退避重试
针对 LLM API 的 529 过载和 429 限流，设计 7 层异常恢复，包括指数退避、Token 超限自动压缩、模型 fallback 降级
# JSON Schema 输出校验
JSON Schema 严格校验模型的结构化输出，解析失败自动触发修复 Prompt
# RAG 本地知识库降级
云端 RAG 检索超时或无结果，自动降级本地 SQLite 向量库，保障基础问答能力不中断
