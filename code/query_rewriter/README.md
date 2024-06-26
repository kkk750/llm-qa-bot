### 实现逻辑：

参考Langchain的prompt_template实现自己的query rewriter，通过lambda独立成一个接口供调用


### 调用方法

```python
# call this lambda from another lambda
from boto3 import client as boto3_client
lambda_client = boto3_client('lambda')

def lambda_handler(event, context):
    question = event['prompt']
    msg = {
      "params": {
        "history": ["有戴森的吹风机吗？","没有哦亲亲", "戴森都没有", "不好意思，看看其他品牌呢"],
        "query": question
      },
      "use_bedrock" : "True"
    }
    invoke_response = lambda_client.invoke(FunctionName="Query_Rewrite",
                                           InvocationType='RequestResponse',
                                           Payload=json.dumps(msg))
    
```


### 测试case

```json
#1
{
  "params": {
    "history": ["有戴森的吹风机吗？","没有哦亲亲", "戴森都没有", "不好意思，看看其他品牌呢"],
    "query": "那有松下的吗？"  
  },
  "use_bedrock" : "True"
}

#2
{
  "params": {
    "history": ["你喜欢周杰伦吗", "我喜欢周杰伦"],
    "query": "你喜欢他哪首歌"
  },
  "use_bedrock" : "True",
  "role_a" : "user",
  "role_b" : "bot"
}

#3
{
  "params": {
    "history": ["Sagemaker相关问题应该联系谁？","李雷"],
    "query": "那Emr的问题呢？"
  },
  "use_bedrock" : "True"
}

#4
{
  "params": {
    "history": ["zero-etl在中国可用了吗？","还不可用", "在global可用了吗？", "可用"],
    "query": "中国区sagemaker有jumpstart吗"
  },
  "use_bedrock" : "True"
}

#5
{
  "params": {
    "history": ["AI的销售是谁？","请问是哪个地区的？"],
    "query": "北区"
  },
  "use_bedrock" : "True",
  "llm_model_name":"claude-instant"
}
```


### 优化手段

1. 速度层面，使用IUR(Incomplete Utterance Rewrite)相关的[小模型](https://huggingface.co/csdc-atl/dialogue-rewriter)进行推理