# 核心
一段提前写好的"指令说明书"，在特定时机塞给大模型看
## 结构
```
skill名称/
├── SKILL.md（必须）
│   ├── YAML 头部元数据（name、description 为必填项）
│   └── Markdown 说明文档
└── Bundled Resources（可选）
    ├── scripts/    - 用于确定性/重复性任务的可执行代码
    ├── references/ - 按需加载到上下文的参考文档
    └── assets/     - 输出中使用的文件（模板、图标、字体等）
```
## 流程
```
用户输入

      ↓                                                                                                                                                                                        
Agent 看描述，判断调用 Skill

      ↓

把 SKILL.md 的完整内容塞进对话上下文            

      ↓

塞给大模型处理 
```

本质是：自动往大模型上下文里插入一段文字，把"特殊要求"固化下来，每次遇到相关场景都自动执行，不用反复交代

```
第1轮请求：Agent 把所有已装载 Skill 的 description 注入 System Prompt，让模型判断是否需要触发某个 Skill
模型返回：模型判定命中，触发 Skill，"合作商规范等" 命中 description 中的触发场景
第2轮请求：Agent 加载完整 SKILL.md，注入 System Prompt，模型按Skill流程先执行检查
模型返回：检查结果和当前状态
第3次请求：在上一轮的 messages 基础上，追加 tool 执行结果
... ... 
```
