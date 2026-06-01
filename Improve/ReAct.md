# 异常恢复
## HTTP 529 & HTTP 429
问题：  
HTTP 529：服务器过载（Service Overloaded）  
HTTP 429：请求过多（Too Many Requests），触发限流  
防止 API 服务被瞬时高并发压垮，保证公平分配资源  

解决：  
API 返回状态码后，客户端检测到 529/429 → 进入重试队列  
限流重试时要遵守服务端返回的 Retry-After 头（秒数）  
没有 Retry-After 头则采用指数退避策略    
要区分硬限流（必须等待）和软限流（可立即重试）  
在分布式客户端中要加抖动，避免同时重试  

原理：  
服务端有负载监控和限流器（Token Bucket / Leaky Bucket）  
当 QPS（Queries Per Second） 超过阈值，拒绝请求并返回状态码  
客户端通过重试策略（固定延迟/指数退避）减少压力
## 指数退避重试
问题：  
所有客户端在同一时间重试，造成雪崩    

解决：  
重试间隔按指数增长：1s → 2s → 4s → 8s → …    

```
import time, random

def retry_with_backoff(func, max_attempts=5):
    delay = 1
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            time.sleep(delay + random.uniform(0, 0.5))  # 加随机抖动
            delay *= 2
```

原理：  
指数退避：每次失败后延迟时间翻倍（延迟 = 基础延迟 × 2^重试次数）  
随机抖动：避免多个客户端同时发起重试（减少同步雪崩）  
常用于分布式系统的冲突解决、API 限流恢复  
## Token 超限压缩
问题：  
当 Prompt + Context Token 数超过模型限制（如 8k、32k） 
超限会直接报错或截断输入，导致上下文丢失  

解决：  
优先裁剪历史对话：删除最早的消息  
摘要压缩：把多轮历史合并成摘要（保留关键信息）  
向量检索：只保留与当前任务相关的上下文  
压缩后要重新计算 Token 数验证  

原理：  
LLM 输入是 Token 序列，超过限制会拒绝处理    
压缩是通过 NLP 摘要算法 / RAG 检索减少 Token 数  
向量检索利用 Embedding 找出相关性高的历史片段
## 模型 Fallback 降级
问题：  
主模型不可用（宕机、升级、限流）   

解决： 
自动切换到备用模型  
在调用前检测模型可用性  
切换逻辑要有超时和重试

```
def call_llm(prompt):
    try:
        return call_primary_model(prompt)
    except ModelUnavailable:
        return call_backup_model(prompt)
```

原理：  
多模型配置：主模型（高质量）、备用模型（便宜/稳定）  
通过健康检查（ping API）判断可用性  
Agent 框架（LangChain、CrewAI）支持多模型路由  
## Tool Calling 超时
问题：  
工具调用超时未响应  

解决：  
设置超时熔断（如5s），未返回自动降级为缓存响应  

原理：  
本地判断超时未响应主动降维本地缓存处理，避免对话卡死  
## ReAct 死循环检测
问题：  
Agent 多轮调用工具时出现死循环（重复调用相同工具、参数相同）  

解决：  
通过 Hash 比对 + 迭代阈值（如果重复率超过阈值 → 触发熔断）双重校验，实时拦截 Tool Calling 死循环  
记录最近 N 次调用的 Hash,检测重复率  

原理：  
防止无限循环消耗 Token 和 API  
类似分布式系统的熔断器（Circuit Breaker）  
## JSON Schema 输出校验
问题：  
模型输出格式不符合预期（缺字段、JSON 解析失败）  

解决：  
JSON Schema 验证输出，如果验证失败 → 把错误信息作为新 Prompt，让模型修正  
验证要在客户端完成,修复要有次数限制（避免死循环）  

原理：  
利用 LLM 自我修复能力（Self-Healing）  
Schema 验证是结构化任务的关键（防止后续处理失败（JSON 解析错误、缺字段））  
## RAG 本地知识库降级
问题：  
API 调用失败、云端 RAG 检索超时或无结果  

解决：  
在工具调用时先查本地缓存，缓存命中 → 返回数据；未命中 → 请求 API，请求失败 → 自动降级本地 SQLite 向量库  

原理：  
Cache-aside 模式：先查缓存，缓存未命中再查源  
提高可用性，减少 API 调用次数，本地数据库不断保存云端返回数据，保障基础问答能力不中断
