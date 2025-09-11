# RAG 进阶学习笔记

首先来理清楚思路:

### 2、RAG 工作流程

#### 1、文档收集和切割

* **文档收集**：从网页、PDF、数据库等各种来源收集原始文档。
* **文档预处理**：清洗、标准化文本格式。
* **文档切割**：将长文档分割成适当大小的片段，可基于固定大小、语义边界、递归分割策略。

这一个流程我们做了什么事情：

准备文档存放在本地资源目录下

之后是开始文档读取，我们用了

DocumentReader:读取文档,得到文档列表，读取 PDF、TXT、JSON 等文件 `**new PdfDocumentReader("file.pdf")**`

DocumentTransformer: 转换文档,得到处理后的文档列表，拆分、加摘要、提关键词 `**new TokenTextSplitter().apply(documents)**`

DocumentWriter: 将文档列表保存到存储中,把结果写入数据库或文件 `**vectorStore.write(documents)**`



#### 2、向量转换和存储

* **向量转换**：使用Embedding模型将文本块转换为高维向量表示，以捕获文本的语义特征。

* **向量存储**：将生成的向量和对应文本存入向量数据库，支持高效的相似性搜索。
  在这个步骤我们将嵌入模型DashScope将文档内容转换为向量,通过VectorStore保存向量及元数据到内存中

工作原理:1文本转向量用到了嵌入模型，2:存入数据库 3: 语义搜索:查询页转为向量,找最相似的内容.

向量数据库的配置VectorStore



#### 3、文档过滤和检索

* **查询处理**：将用户问题转换为向量表示。
* **过滤机制**：基于元数据、关键词或自定义规则进行过滤。
* **相似度搜索**：在向量数据库中查找与问题向量最相似的文档块，常用算法有余弦相似度、欧氏距离等。
* **上下文组装**：将检索到的多个文档块组装成连贯上下文。
  
  

#### a. 预检索：优化用户查询

* **改写查询**：用 AI 让模糊的问题更清晰`RewriteQueryTransformer`
* **翻译查询**：将非目标语言翻译成模型支持的语言`TranslationQueryTransformer`
* **压缩查询**：结合对话历史，生成简洁查询`CompressionQueryTransformer`
* **扩展查询**：生成多个变体，提高召回率`MultiQueryExpander` 补充：什么是招回,就是多搜集3到5个跟他相关有联系的内容或者文档.
  
  

#### b. 检索：查找相关文档

  使用DocumentRetriever 从向量库中搜索最相关的文档，它是抽象接口

* 支持设置：

* 相似度阈值 `.similarityThreshold(0.7)`

* 返回数量 `.topK(5)`

* 元数据过滤 `.filterExpression(...)`

* **多源合并**：使用 `ConcatenationDocumentJoiner` 合并多个结果
  
  

## c. 检索后：优化结果

* 对检索到的文档进行：

* 排序（按相关性）

* 精简（去重、删冗余）

* 压缩（减少上下文长度占用）
  
  
  
  

#### 4、查询增强和关联

* **提示词组装**：将检索到的相关文档与用户问题组合成增强提示。
* **上下文融合**：大模型基于增强提示生成回答。
* **源引用**：在回答中添加信息来源引用。
* **后处理**：格式化、摘要或其他处理以优化最终输出。

我们运用到了查询增强,通过QuestionAnswerAdvisor拦截器增强问答流程,用户提问时,拦截器检索向量数据库获取相关文档切片。将检索结果拼接至用户问题。 直接在chatClient里面调用的

.advisors(new QuestionAnswerAdvisor(loveAppVectorStore))

如果这一步用的是云知识库检索,首先配置云知识库检索,之后同理

.advisors(loveAppRagCloudAdvisor)



#### a. 查询增强目标：

* 提高用户查询质量
* 增加检索命中率
* 提供上下文，辅助 AI 生成更准确回答

#### b. 核心组件：

##### ⅰ. `QuestionAnswerAdvisor`

* 将用户问题 + 检索到的文档拼成新 Prompt，发给 AI。

* 支持设置：

* 相似度阈值 `.similarityThreshold()`

* 返回数量 `.topK()`

* 动态过滤条件 `.FILTER_EXPRESSION`

* 可自定义提示词模板。
  
  

##### ⅱ. `RetrievalAugmentationAdvisor`

* 更灵活、模块化的 RAG 实现方式。

* 支持组合使用：

* 文档检索器 `.documentRetriever(...)`

* 查询转换器（如改写、翻译）`.queryTransformers(...)`

* 上下文增强器 `.queryAugmenter(...)`
  
  

##### ⅲ. 空上下文处理：`ContextualQueryAugmenter`

* 默认不允许空上下文（无文档时不让回答）

* 可通过 `.allowEmptyContext(true)` 允许 AI 自由作答

* 支持自定义提示词模板，包括：

* 正常情况 `.promptTemplate(...)`

* 无文档时 `.emptyContextPromptTemplate(...)`（如友好提示）
  
  

# 具体的场景，知识点全部串起来:

------------------

![c8ec1720-ea6a-43a6-ac43-26465b43e7b5](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/c8ec1720-ea6a-43a6-ac43-26465b43e7b5.png)

![b1cc8651-fdb6-42cf-b0e2-44102e41e8b5](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/b1cc8651-fdb6-42cf-b0e2-44102e41e8b5.png)

![4da269bd-7c09-4fe5-b493-0588f609d9ad](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/4da269bd-7c09-4fe5-b493-0588f609d9ad.png)

![ac769c3d-9143-45f2-8511-438afeaa5345](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/ac769c3d-9143-45f2-8511-438afeaa5345.png)

![65a083f9-3cca-4998-8615-d492799d382e](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/65a083f9-3cca-4998-8615-d492799d382e.png)

![6ebc09a9-081e-46d5-beaa-72feeea3fd1e](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/6ebc09a9-081e-46d5-beaa-72feeea3fd1e.png)

![2445c3db-40f4-4712-a6d4-72f9251e9238](file:///C:/Users/79938/OneDrive/%E5%9B%BE%E7%89%87/Typedown/2445c3db-40f4-4712-a6d4-72f9251e9238.png)



现在回到我们的项目中: 首先文档收集和切割

主要流程ETL：

在T中我们加了切片功能,还添加了关键词增强（MyKeywordEnricher）：添加了元信息,作用:RAG 可以：

* 用向量相似度找到内容接近的文档。

* 用 metadata（关键词）过滤掉不相关的文档。 
  
  

文档收集和切割的任务结束,下面是向量转换和存储

// 创建PgVectorStore实例 

把文档加载进来，之后存进向量数据库中,里面有一些关键配置项要配置：例如维度，计算举例的方法.

向量转换和存储的任务结束，下面是文档过滤和检索



#### a. 预检索：优化用户查询

* **改写查询**：用 AI 让模糊的问题更清晰`RewriteQueryTransformer`

* **翻译查询**：将非目标语言翻译成模型支持的语言`TranslationQueryTransformer`

* **压缩查询**：结合对话历史，生成简洁查询`CompressionQueryTransformer`

* **扩展查询**：生成多个变体，提高召回率`MultiQueryExpander`  在项目中构建了多查询,之后写了一个 执行查询重写的方法，在把用户的问题发给大模型之前进行扩展了。
  String rewrittenMessage = queryRewriter.doQueryRewrite(message);  
  ChatResponse chatResponse = chatClient  
  
          .prompt()  
          // 使用改写后的查询  
          .user(rewrittenMessage)

#### b. 检索：查找相关文档

* 使用 `DocumentRetriever` 从向量库中搜索最相关的文档

创建文档检索器：

```xml
DocumentRetriever documentRetriever = VectorStoreDocumentRetriever.builder()
            .vectorStore(vectorStore)
            .filterExpression(expression) // 过滤条件
            .similarityThreshold(0.5) // 相似度阈值
            .topK(3) // 返回文档数量
            .build();

```



#### c. 检索后：优化结果

* 对检索到的文档进行：

* 排序（按相关性）

* 精简（去重、删冗余）

* 压缩（减少上下文长度占用）



### 4. 🧩查询增强和关联

#### a. 查询增强目标：

* 提高用户查询质量
* 增加检索命中率
* 提供上下文，辅助 AI 生成更准确回答



#### b. 核心组件：

##### ⅰ. `QuestionAnswerAdvisor`

* 将用户问题 + 检索到的文档拼成新 Prompt，发给 AI。

* 支持设置：

* 相似度阈值 `.similarityThreshold()`

* 返回数量 `.topK()`

* 动态过滤条件 `.FILTER_EXPRESSION`

* 可自定义提示词模板。

##### ⅱ. `RetrievalAugmentationAdvisor`

* 更灵活、模块化的 RAG 实现方式。

* 支持组合使用：

* 文档检索器 `.documentRetriever(...)`

* 查询转换器（如改写、翻译）`.queryTransformers(...)`

* 上下文增强器 `.queryAugmenter(...)`

##### ⅲ. 空上下文处理：`ContextualQueryAugmenter`

* 默认不允许空上下文（无文档时不让回答）

* 可通过 `.allowEmptyContext(true)` 允许 AI 自由作答

* 支持自定义提示词模板，包括：

* 正常情况 `.promptTemplate(...)`

* 无文档时 `.emptyContextPromptTemplate(...)`（如友好提示）



总结:

1. 检索 (Retrieval)

-----------------

在 RAG 里，**检索的意思是：从你提供的知识库里找出和用户问题最相关的文档块**。



2. 查询增强 (Query Augmentation)

----------------------------

**查询增强不是检索，它是“拼装 Prompt”这一步**。  
也就是：

* 把用户问题 + 检索到的文档 **组合成一个新的 Prompt**

* 然后交给大模型生成回答

👉 **查询增强的目标** = 给 AI 更多上下文，让它回答得更准确。



RAG 流程拆解
--------

1. **前处理（Pre-processing / 检索增强）**
   
   * 你现在做的：
     
     * 文档切片 → 存入向量库
     
     * 检索 → 找到相关文档
     
     * 查询增强 → 把用户问题 + 文档拼成 Prompt
   
   * 目标：**让 AI 拿到最优质的输入**。

2. **生成（Generation / AI 回答）**
   
   * LLM 根据 Prompt 生成回答。

3. **后处理（Post-processing / 输出增强）**
   
   * 这一步很多人忽视，其实很有用：
     
     * **引用增强**：给回答加上出处（文档链接、段落 ID）。
     
     * **答案过滤**：做安全检查，避免输出敏感/有害信息。
     
     * **答案压缩**：如果回答太长，可以再摘要一遍。
     
     * **结构化输出**：比如用户要 JSON、表格格式，而不是一堆自然语言。
     
     * **打分/重排序**：如果多个候选答案，可以用另一个模型做质量评估，选最优。
