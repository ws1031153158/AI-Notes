# 背景
早期（比如 GPT-3.5 刚推出时），大模型只能输出文本。我们需要通过复杂的 Prompt 诱导模型输出 JSON，然后手动解析
# 标准化
大模型厂商可以搞事情了，直接把“调用Function”这件事情标准化到模型内部  
模型对 Tool 的调用动作，就叫 Function Call   

```
{
  /* 第一层：工具类型声明 */
  "type": "function", 
  
  /* 第二层：函数核心定义 */
  "function": {
    /* 1. 唯一标识符：模型调用时返回的名称，建议用下划线命名 */
    "name": "xxx_name",
    
    /* 2. 语义描述（核心）：模型靠这段文字判断“应不应该调我”。
       建议包含：功能、适用场景、前置条件。 */
    "description": "",
    
    /* 第三层：参数约束（JSON Schema） */
    "parameters": {
      "type": "object",
      "properties": {
        /* 定义具体的参数项 */
        "prop1": {
          "type": "string",
          "description": "参数1的描述"
        },
        
        "prop2": {
          "type": "boolean",
          "description": "参数2的描述",
          "default": false
        },
        ...
      },
      /* 必填项：模型如果不提供这些参数，将无法触发调用 */
      "required": ["prop1", "prop2"]
    }
  }
}
```
## Tool Schema
```
{
{
  "name": "searchVideos",
  "description": "Search videos by keyword",
  "parameters": {
    "type": "object",
    "properties": {
      "keyword": { "type": "string", "description": "Search keyword" },
      "limit": { "type": "integer", "description": "Max number of results" }
    },
    "required": ["keyword"]
  }
}
```

在使用 Tool 时，模型进入了 "Call Mode":  
1.输入包装：在 API 调用时，不把格式要求写在 content 里，而是传一个独立的 tools 参数  
2.输出处理：当模型匹配到这个 Tool 时，会触发一个特殊的 Finish Reason: tool_calls。此时，模型内部的 Token 生成路径会被截断，直接跳过所有自然语言生成的概率分支，只输出 JSON 字符串  

## Invocation
```
{
  "tool": "searchVideos",
  "arguments": {
    "keyword": "TikTok dance",
    "limit": 5
  }
}
```
## Response
```
{
  "tool": "searchVideos",
  "output": [
    { "title": "Dance 1", "url": "..." },
    { "title": "Dance 2", "url": "..." }
  ]
}
```
