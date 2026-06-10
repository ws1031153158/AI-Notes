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
# 如何写好一个 Skill
1.边界明确  
2.输入输出结构化  
3.步骤明确可执行  
4.失败策略完备  
5.单一职责
# Skill Creator
## 创建 Skill
1. 捕获意图（Capture Intent）  
从对话历史中提取信息（用了哪些工具、步骤顺序、用户纠正了什么）  
要确认四个核心问题：  
这个 skill 让 大模型 能做什么？   
什么时候触发？  
输出格式是什么？  
是否需要测试用例？  
特点：客观可验证的 skill（文件转换、代码生成）建议加测试用例；主观类（写作风格、艺术）通常不需要  

2. 访谈与调研（Interview and Research）    
主动提问边界情况、输入输出格式、成功标准、依赖关系  
如果有 MCP 工具可用，通过并行子Agent同时调研  
特点：先把问题搞清楚再写 test prompt，降低用户负担  

3. 编写 SKILL.md    
必须包含：name、description（触发机制）、具体指令  
description 的特别设计：要"积极主动（pushy）"——大模型有低触发倾向，所以描述要明确列出各种触发场景

4.Skill书写指南  
```
skill-name/
├── SKILL.md          # 必须
└── Bundled Resources/ # 可选
    ├── scripts/      # 可执行脚本（无需加载即可运行）
    ├── references/   # 按需加载的文档
    └── assets/       # 模板、图标等
```
## 运行与评估测试用例
1.同一轮次同时启动所有测试（with-skill + baseline）  
对每个测试用例，同时启动两个子代理：  
with_skill：带 skill 运行  
without_skill（新建 skill）或 old_skill（改进已有 skill）：基线对比  
特点：必须同时启动，不能先跑有 skill 的再回来跑基线  

2.运行期间起草 Assertions（断言）  
利用等待时间撰写量化断言，而不是干等  
断言要求：客观可验证、名称描述性强（一眼看懂在测什么）  
主观 skill（写作风格等）不要强行加断言，适合定性评估  
更新 eval_metadata.json 和 evals/evals.json  

3.子代理完成时立刻记录 timing 数据  
子代理完成的通知中包含 total_tokens 和 duration_ms  
必须立即保存到 timing.json（这是唯一的捕获机会）  

4.打分 → 聚合 → 分析 → 启动可视化 Viewer    
打分：使用 agents/grader.md 中的规范评估每条断言，能写脚本就写脚本（更可靠）  
聚合基准：运行 scripts/aggregate_benchmark 生成 benchmark.json，含通过率、时间、token 数及标准差  
分析师过审：识别"无论有没有 skill 都能通过"的无效断言、高方差（可能不稳定）的测试等  
启动 Viewer：  
Outputs 标签：查看每个测试用例的输出，可留反馈  
Benchmark 标签：查看量化对比统计  
无浏览器环境：用 --static 生成静态 HTML  
## 迭代与触发优化
根据用户反馈和量化数据重写 skill  
多轮后扩大测试集规模再验证  
最后可运行 description improver 脚本专门优化 skill 的触发准确性  
# 校验
1.C1：缺少 IRON LAW  

```
> **IRON LAW**: NEVER [具体禁止行为]. ALWAYS [对应正确行为].
```

2.C2：缺少 Pre-Delivery Checklist  

```
## 交付前检查清单

⛔ BLOCKING：以下全部打勾后才能输出：

- [ ] 输出内容完整（无截断）
- [ ] 无硬编码个人信息
- [ ] 引用文件均存在
- [ ] [根据 skill 类型添加具体项]
```

3.C3：缺少 Anti-Pattern 列表  

```
## Anti-Pattern 列表

| # | 禁止行为 | 原因 |
|---|---------|------|
| 1 | [具体禁止行为] | [原因] |
| 2 | [具体禁止行为] | [原因] |
| 3 | [具体禁止行为] | [原因] |
```

4.C4：缺少 Eval 测试  

```
[
  {
    "prompt": "真实用户触发词",
    "expected_behavior": "预期 agent 行为描述",
    "should_trigger": true
  },
  {
    "prompt": "不应触发的 prompt",
    "expected_behavior": "不激活此 skill",
    "should_trigger": false
  }
]
```

5.C5：改动类操作缺少 Confirmation Gate  

```
## ⛔ Confirmation Gate

执行前向用户展示：
- 目标：[路径/群ID/文档ID]
- 操作：[具体会做什么]
- 内容摘要：[前100字]

**等待用户明确确认后再执行。**
```

6.C6：阶段无出口条件  

```
**出口条件**：[可 yes/no 判断的完成标准，例如"SQL 执行成功且返回结果不为空"]
```

7.C8：硬编码个人信息  

```
✗  "misId": "chenzhiguang02"  →  ✓  "misId": "<your_mis_id>"
✗  "session": "abc123xyz"     →  ✓  "session": "<your_session_id>"
✗  "appKey": "APP_KEY_XXXXX"  →  ✓  "appKey": "<your_app_key>"
```

8.C9：description 不完整  

```
description: "[用途一句话]。触发词：[触发词列表]。使用条件：[何时使用]。跳过条件：[何时不使用]。"
```
