# AI应用开发

## 笔记

Prompt ﻿工程（Prompt Eng‏ineering）又叫提示‌词工程，简单来说，就是输入​给 AI 的指令。

Token 是﻿大模型处理文本的基本单位，可‏能是单词或标点符号，模型的输‌入和输出都是按 Token ​计算的，一般 Token 越​多，成本越高，并且输出速度越慢。

如何计算 Token？首先，不同﻿大模型对 Toke‏n 的划分规则略有‌不同，比如根据 O​penAI 的文档​：

* 英文文本：一个 token 大约相当于 4 个字符或约 0.75 个英文单词
* 中文文本：一个汉字通常会被编码为 1-2 个 token
* 空格和标点：也会计入 token 数量
* 特殊符号和表情符号：可能需要多个 token 来表示

Pro﻿mpt 优化相关的资源：

* OpenAI 提示词工程指南：[OpenAI Platform](https://platform.openai.com/docs/guides/prompt-engineering)
* Spring AI 提示工程指南：[Prompts :: Spring AI Reference](https://docs.spring.io/spring-ai/reference/api/prompt.html#_prompt_engineering)
* 智谱 AI Prompt 设计指南：[智谱AI开放平台](https://open.bigmodel.cn/dev/guidelines/LanguageModels)

MVP 最小可行产品策略是指先开发包含**核心功能**的基础版本产品快速推向市场，以最小成本验证产品假设和用户需求。通过收集真实用户反馈进行迭代优化，避免开发无人使用的功能，降低资源浪费和开发风险。

Spring AI 使用 Advisors（顾问）机制来增强 AI 的能力，可以理解为一系列**可插拔的拦截器**，在调用 AI 前和调用 AI 后可以执行一些额外的操作，比如:

               1. 前置增强：调用 AI 前改写一下 Prompt 提示词、检查一下提示词是否安全    

               2. 后置增强：调用 AI 后记录一下日志、处理一下返回的结果

## 系统预设提示词

Text:  ```我正在开发【健身大师】AI 对话应用，编写设置给 AI 大模型的系统预设 Prompt 指令。```
    1:对话风格
    2:咨询流程
    3:输出要求
    4:首次对话引导

## 多轮对话AI应用开发

1.新建App包,新建FitnessApp类 构造ChatClient对象.

   ChatClient本身是对话入口,记忆是通过Advisor插件化加入的,同时存储在内存里也是调用这个.               

2.添加doChat方法: 接受用户输入的文本和对话ID,用`chatClient.prompt()` 创建一次对话.

3.编写测试方法,测试多轮对话. 新建FitnessAppTest类, 多轮对话,测试记忆是否生效.



## 自定义日志Advisor类

1.新建advisor包,新建MyLoggerAdvisor类,@Slf4j注解:直接用log.info(),log.error()来打日志.

   自定义日志,挂在Spring AI 的对话调用链上,用来:

        1: 在请求前打印用户输入(Prompt).

        2: 在响应后打印AI的回复(只取输出文本).

        3: 支持同步调用(aroundCall), 也支持流式调用(aroundStream).

 MyLoggerAdvisor implements CallAroundAdvisor, StreamArundAdvisor 

 implements表示实现接口,必须去实现接口里声明的方法. 

CallAroundAdvisor, StreamArundAdvisor 这两个接口规定了必须有的方法：

* `aroundCall(...)`

* `aroundStream(...)`

* `getName()`   返回类作为Advisor的名字

* `getOrder()`    设置执行顺序

2.将自定义的日志Advisor加入到默认Advisor中



## 格式化AI输出

1. 新增依赖,定义数据模型,将AI输出转换为数据模型对象

2. 修改FitnessApp类,在FitnessApp定义AI输出的数据模型FitnessReport(record类型) ,并将AI输出转换为FitnessReport对象.  
   比如要输出标题,之后下面是建议,那么写法: record FitnessReport(String title, List<String> suggestions)
   因为是把AI的输出答案拦截，所以这次用了.call()真正调用大模型API,把消息发出去.
   .entity(FitnessReport.class) 把大模型的回复内容,自动转换为你定义的java类实例.  

3. 编写测试方法.
   
   

## 对话记忆持久化

1. 自定义文件持久化ChatMemory, 新增kryo依赖.

2. 新建chatmemory包,新增编写基于文件持久化的对话
   
   1. FileBasedChatMemory implements ChatMemory 
      
      getConversationFile(String conversationId) : 根据会话 ID，生成一个对应的文件路径
      
      readConversation(File file): 从文件中读取并反序列化消息列表(List<Meassage>), **一句话总结**：**把文件里的会话内容读出来还原成消息列表。**
      
      saveConversation(File file, List<Message> messages): 把传入的消息列表序列化，并写入到指定文件中. 把会话内容保存到文件里，持久化存储。

             根据上面三个接下里可以完成基本的增删改查功能.

   3. 修改 FitnessApp类, 在构造ChatClient时使用自定义的ChatMemory类

      private static final String CHAT_MEMORY_PATH = System.getProperty("user.dir") + "/chat-memory";
      
      .defaultAdvisors(   
                        new MessageChatMemoryAdvisor(
                              new FileBasedChatMemory(CHAT_MEMORY_PATH)
                        )
                 )



## PromptTemplate模板

### 一句话概括把系统提示词写成“模板”，然后在不同场景里自动替换内容，让 AI 按你想要的角色去回答。



## 多模态

1. 添加依赖, 新建demo包,新建MultiModelConversationCallDemo类，
   
   代码参考阿里云百炼文档: https://help.aliyun.com/zh/model-studio/use-qwen-by-calling-api#15ef0a40798a3

        


