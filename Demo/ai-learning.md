# 1
## Prompt
Prompt要清晰，一个完整的Prompt通常包含：  
角色（Role） - 你是谁  
背景（Context） - 情况是什么  
任务（Task） - 要做什么  
格式（Format） - 输出什么样  
## Temperature
越小越保守，越大越开放，根据场景和业务需求确定
# 2
## System Prompt
角色设定，明确职责、制定回答规则，确定边界，要有防御性设计（如回答边界）
# 3
## 结构化输出
约束回答格式：markdown、json等，可固定输出格式，例如Open AI的JSON Mode
# 4
## Chain of Thought 
思维链，区别于直接回答，可以看到每一步的思考过程  
"让我们一步步思考" 这句话有神奇的效果
## Zero-shot CoT
零样本的思维链，让LLM自己生成思考过程，一般两次提示（生成推理过程 -> 提取具体答案）  
无需示例预训练，直接通过提示词让模型生成推理链，更加灵活开放
# 5
## Few-shot
少样本学习，用例子教会AI做特定格式的任务
# 6
## Streaming
流式输出，数据生成的同时被逐步发送或处理，而非等待全部生成后一次性输出，让AI应用有更好的用户体验，为后续项目做准备（stream=True流式 + flush=True实时输出）
## Token 用量统计
prompt_tokens（输入） + completion_tokens（输出）
## 错误处理与重试机制
必须有完善的错误处理，例如：触发限流、请求超时、API错误、未知错误等
