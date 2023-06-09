## LLM 模块介绍

Byzer-LLM Finetune/Deploy 大模型主要通过 LLM 模块。 比如部署一个模型的典型示例如下：

```sql
-- 全局环境参数
!python conf "rayAddress=127.0.0.1:10001";
!python conf "pythonExec=/home/winubuntu/miniconda3/envs/byzerllm-desktop/bin/python";
!python conf "dataMode=model";
!python conf "runIn=driver";

!python conf "num_gpus=0.4";
!python conf "maxConcurrency=1";
!python conf "standalone=true";

!python conf "schema=file";

-- 允许该模块需要的参数
run command as LLM.`` where 
action="infer"
and pretrainedModelType="chatglm"
and localPathPrefix="/my8t/byzerllm/jobs"
and localModelDir="/my8t/byzerllm/jobs/checkpoint-17000/pretrained_model"
and modelWaitServerReadyTimeout="300"
and udfName="chat"
and modelTable="command";
```

和标准SQL 语句很像，运行 LLM 模块，然后参数包含两部分：

1. 全局环境变量，使用 !python conf 来配置。
2. 模块参数，放在where条件后面。

在这段代码里，你只需要修改  `localModelDir` 指向一个正确的chatglm模型地址，就能部署该模型。

下面我们详细介绍上面的参数。

### LLM 参数介绍

#### 全局环境变量

| Parameter | Description |
|--|--|
|rayAddress| Byzer Finetune 和 部署都需要 Ray。这里填写 Ray的地址|
|pythonExec| 指定 Byzer 和 Ray 的 Python 环境|
|dataMode| 可选值 model/data. 在 Byzer-LLM 中一律指定为 model  |
|runIn| 指定 ray client 运行在 Byzer 的driver端还是 executor端。建议 driver端。可选值： driver/executor |
|num_gpus| 指定部署的模型实例需要的GPU资源。注意这里是指一个模型实例需要的GPU资源。可以是小数，这样可以方便多个模型可以调度到一个GPU上 |
|maxConcurrency| 部署多少个模型实例。如果你有多个GPU，可以部署多个实例，从而获得更好的并发 |
|standalone| 如果你是单机部署 Ray, 那么可以将该值设置为true。 |
|schema| 在 Byzer-LLM 中设置为 file 即可 |

#### 通用参数

| Parameter | Description |
|--|--|
|`action="infer"`| finetune 还是 部署模型。可选值为： infer/finetune |
|`pretrainedModelType="chatglm"`| 模型类型。 可选值：chatglm,moss,bark,whisper,dolly,qa,falcon,sass/chatglm,llama|
|`localPathPrefix="/my8t/byzerllm/jobs"`  | 临时文件目录。部署模型的 worker 会产生很多临时文件，需要指定一个目录进行存储，防止默认 /tmp 太小的问题|
|`modelWaitServerReadyTimeout="60"`| 单位秒。Byzer-LLM 会提供模型的 socket server ,这里可以设置为等待socket server ready的时间 建议修改成 300|
|`dataWaitServerReadyTimeout="60"`| 单位秒。Byzer-LLM 会提供数据的 socket server ,这里可以设置为等待socket server ready的时间 建议修改成 300|
|`modelTable="d_chatglm_6b_model"`| Byzer-LLM 将模型也抽象成表，这里指定模型表的名称|
|`localModelDir="/my8t/byzerllm/jobs/checkpoint-17000/pretrained_model"`| 指定worker 本地的模型，这样可以极大的加速模型的加载|

注意1：modelTable 和 localModelDir 两个参数本质都是指定模型文件在哪里。如果都被配置了，会优先使用 localModelDir。
modelTable 需要使用模型分发，速度较慢。如果单机的话，建议使用 localModelDir 参数。当使用 localModelDir的时候，此时将 modelTable 参数配置成 `command`。

注意2： 对于 大模型 SaaS 服务，部署时需要将 num_gpus 设置为 0 (通过命令： !python conf "num_gpus=0"; 来完成), 避免占用 GPU 资源。

#### 部署参数

| Parameter | Description |
|--|--|
|`udfName="origin_model_predict"`|Byzer-LLM 会将模型调用抽象成 SQL 函数，这里可以随意取一个名字，方便后续调用|

sass/chatglm 独有参数：

| Parameter | Description |
|--|--|
|`apiKey="xxxx"`| ChatGLM Saas 服务的 apiKey |
|`publicKey="xxxx"`| ChatGLM Saas 服务的 publicKey | |


#### 微调参数：

| Parameter | Description |
|--|--|
|`learningRate="5e-5"`|学习率|
|`maxSteps="100"`|最大迭代步数|
|`saveSteps="50"`| 每多少步数保存一次checkpoint|

chatglm专属：

| Parameter | Description |
|--|--|
|`quantizationBit="false"`|是否进行量化. This option is only works for ChatGLM. The Moss will auto detect according the model files|
|`quantizationBitNum="8"`|量化位 可选 8/4|
|`finetuningType="lora"`| action=finetune 时，指定 fintune的类型。默认为 lora。目前仅支持 chatglm 的fintune, 可选值： p_tuning/freese/lora|



   