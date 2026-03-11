# LLM
## 本质上做什么
<img width="970" height="226" alt="image" src="https://github.com/user-attachments/assets/e46d6b0f-c899-4930-a335-5b07078d55f8" />

LLM 本质上是一个超级复杂的"下一个词预测器"，它在海量文本上训练，学会了"什么词后面跟什么词最合理"，但因为训练数据足够多，它"涌现"出了理解、推理、写作的能力。  
普通输入法：  "今天天气" → 预测 "不错"  
LLM：        "今天天气" → 预测出一篇天气分析报告  
本质是同一件事，但LLM的参数量是输入法的千万倍  
## Training & Inference
### Training
海量数据调整模型参数，巨量GPU
### Inference
用训练好的模型处理你的输入，生成输出，调用API或本地运行模型  
## 参数量
7B   = 70亿参数   → 可以本地跑，效果一般  
14B  = 140亿参数  → 本地跑需要好显卡，效果较好  
72B  = 720亿参数  → 本地跑需要多卡，效果很好  
GPT-4 → 参数量未公开，估计超过1万亿  
# Token
模型处理文本的最小单位。  
影响费用、上下文窗口（ContextWindow）、系统设计（RAG）
# ContextWindow
模型在一次对话中能"看到"的最大Token数量，也就是短期记忆容量，超出这个范围的内容，模型完全不知道存在
## 内容
<img width="910" height="1028" alt="image" src="https://github.com/user-attachments/assets/1b73d30b-db8e-431d-b24f-80382f6ad772" />

## 影响
<img width="968" height="668" alt="image" src="https://github.com/user-attachments/assets/ca3144ce-093f-4444-8576-d62c2e83ae33" />

# RAG
## Embedding
