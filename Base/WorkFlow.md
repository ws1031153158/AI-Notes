# Workflow vs Agent
## Workflow
LLM 和工具通过预定义代码路径进行编排的系统  
由代码决定  
## Agent
LLM 动态决定自身处理流程和工具使用的系统  
由模型决定
## Agentic
1.外层用 Workflow 控制整体流程，保障确定性和可审计性  
2.内层用 Agent 处理需要灵活性的子任务（如编码、搜索、分析）  
3.设置 Guardrails：超时限制、token 上限、循环检测、输出校验  
4.Human-in-the-loop：关键决策保留人工确认  
5.可观测性：每一步有日志、指标、可追溯  
