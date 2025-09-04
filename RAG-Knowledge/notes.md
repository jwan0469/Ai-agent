

# RAG知识库基础

## 笔记

**RAG**（**Retrieval-Augmented Generation，检索增强生成**）是一种结合信息检索技术和Al内容生成的混合架构，可以解决大模型的知识时效性限制和幻觉问题；

RAG技术实现主要包含以下4个核心步骤：**文档收集和切割、向量转换和存储、文档过滤和检索、查询增强和关联**；

文档收集和切割：

1. 文档收集：从各种来源收集原始文档；
2. 文档预处理：清洗、标准化文本格式；
3. 文档切割：基于固定大小、语义边界、递归分割等策略，将长文档分割成适当的片段（也称chunks）；

向量转换和存储：

1. 向量转换：使用 Embedding 模型将文本块转换为高维向量；
2. 向量存储：将生成的向量和对应文本存入向量数据库，支持高效的相似性搜索；

文档过滤和检索：

1. 查询处理：将用户问题也转换为向量表示；
2. 过滤机制：基于元数据、关键词或自定义规则进行过滤；
3. 相似度搜索：在向量数据库中查找与用户问题向量最相似的文档块，常用的相似度搜索算法：余弦相似度、欧式距离等；
4. 上下文组装：将检索到的多个文档块组装成连贯上下文；

查询增强和关联：

1. 提示词组装：将检索到的相关文档与用户问题组合成增强提示；
2. 上下文融合：大模型基于增强提示生成回答；
3. 源引用：在回答中添加信息来源引用；
4. 后处理：格式化、摘要或其他处理已优化最终输出；

RAG 检索增强生成-**完整工作流程**：

![7b899ae03738c53c4f223f164608ca0](C:\Users\79938\OneDrive\文档\WeChat%20Files\wxid_h6flgzuz6rdr22\FileStorage\Temp\7b899ae03738c53c4f223f164608ca0.png)

**Embedding 嵌入**是将高维离散数据（如文字、图片）转换为低维连续向量的过程。这些向量能在数学空间中表示原始数据的语义特征，使计算机能够理解数据间的相以性；

**Embedding 模型**是执行这种转换算法的机器学习模型，如 Word2Vec (文本)、ResNet (图像)等。不同的 Embedding 模型产生的向量表示和维度数不同，一般维度越高表达能力更强，可以捕获更丰富的语义信息和更细微的差别，但同样占用更多存储空间。

**向量数据库**是专门存储和检索向量数据的数据库系统。通过高效索引算法实现快速相似性搜索，支持K近邻查询等操作；

**召回**是信息检索中的第一阶段，目标是**从大规模数据集中快速筛选**出可能相关的候选项子集。**强调速度和广度，而非精确度**。好比在图书馆里面找书,不是精挑细选，而是把所有跟这个主题有关的书都拿过来

**精排**（**精确排序**）是搜索/推荐系统的最后阶段，考虑更多特征和业务规则，对少量候选项进行更复杂、精细的排序。加入一些特征(features)例如与AI相关的书,这本书是否是经典教材,其他读者是否经常借用这本书.

**Rak模型**（**排序模型**）负责对召回阶段筛选出的候选集进行精确排序. 现代Rank模型通常基于深度学习，如BERT、LambdaMART等，综合考虑查询与候选项的相关性、用户历史行为等因素. 用一个打分器为每本侯选书打分,其实就是评分+排序的过程.

**混合检索策略**结合多种检索方法的优势，提高搜索效果。常见组合包括关键词检索、语义检索、知识图谱等。

不同检索方法组合起来,推荐结果更全面准确.



## RAG实战: Spring AI + 本地知识库

### 准备本地知识库

1. 准备了三篇markdown文件,(AI帮我生成), 文章的问题分别针对三类不同的人群,每个问题标题使用4级标题, 这样可以按标题来进行文档切割.  markdown后缀名为.md文件

2. 新建document包,把三个md文件复制到document包下,新增依赖.

3. 新建rag包,自定义文档加载器类AppDocumentLoader,负责读取所有的markdown文件并转换为Document列表.
   总结: 这个类就是一个 Markdown 文件批量读取工具，把指定目录下的所有 `.md` 文件解析成带元信息的 `Document` 列表，为后续 RAG 检索用。 
   **什么是元信息?** 就是给片段打上标签, 知道片段属于哪个文件, 哪一部分，以后能做过滤(比如"只搜健身笔记").

       **为什么要把Markdown转换成Document列表?**  让文档结构化（内容 + 元信息），方便后续切分、存储、检索和问答，而不是死板的一堆字符串。 

* 如果你直接读字符串，就像是把整本书 **一股脑丢给你**，你还得自己去翻页、找标题。

* 转成 `Document` 对象，就像把书分成一张张卡片，每张卡片写清楚内容和出处（第几页、哪本书），方便快速检索和引用。
  **代码重点**: 

```xml
1.获取资源:
Resource[] resources = resourcePatternResolver.getResources("classpath:document/*.md");

2.文件读取
MarkdownDocumentReader markdownDocumentReader = new MarkdownDocumentReader(
    resource,
    MarkdownDocumentReaderConfig.builder
        .withHorizontalRuleCreateDocument(true)遇到 Markdown 里的分割线（---）就拆分成多份文档。
        .withAdditionalMetadata("filename", resource.getFilename()) //加上元信息
        .build()
);

3.收集文档
 调用 markdownDocumentReader.get() 得到一个 List<Document>，累加到 allDocuments 里。

4.返回结果
    最后返回所有的Dounment对象.
```

   这里回顾个知识点避免混淆: 

   @**Compoent** 用法特点: 类本身就是Bean, 帮我注册进去。

   例子：

            @Component 

            public class AppDocumentLoader {}



   **@Configuration 用途**: 定义配置类,里卖写方法来生成Bean. 用法特点:一般配合@Bean返回对象。这个类是工厂,我会在里面造一些Bean, 帮我注册进去.  

   例子:

       @Configuration
        public class LoveAppVectorStoreConfig {
                @Bean
                public VectorStore vectorStore() {
                        return new SimpleVectorStore(...);
                }
        }

4. 新建AppVectorStoreConfig配置类,构造一个VectorStore对象并加入Spring容器
   代码功能: 用指定的 Embedding 模型，把 `AppDocumentLoader` 加载到的 Markdown 文档转成向量，并存进内存向量库 SimpleVectorStore，供后续检索使用。

```
/**
 * 向量存储配置类
 * 构造一个 VectorStore 对象，用于存储和检索向量数据
 */
@Configuration
public class AppVectorStoreConfig {

    @Resource
    private AppDocumentLoader appDocumentLoader;

    @Bean
    VectorStore appVectorStore(EmbeddingModel dashscopeEmbeddingModel) {
        SimpleVectorStore simpleVectorStore =             SimpleVectorStore.builder(dashscopeEmbeddingModel).build();
        //这一步完成了把切割好的文档文件变成了向量,并存储到了向量数据库里.
        simpleVectorStore.add(appDocumentLoader.loadMarkdowns());
        return simpleVectorStore;
    }
}
```

5. 修改FitnessAPP类,新增doChatWithRadLocal方法:
    .advisors(new QuestionAnswerAdvisor(appVectorStore)

6. 新增测试方法
   
   

### RAG实战: SpringAI + 云知识库

### 1. 准备云知识库

    1. 这里我还是用阿里云百炼, 利用云知识库完成文档读取,文档处理,文档加载,保存到向量数据库,知识库管理等操作. 缺点是额外的费用，以及数据隐私问题.

### 2. RAG开发

    1. 构造 RetrievalArgumentationAdvisor,新建 AppRagCloudAdvisor 配置类， 构造一个 Advisor 对象并加入 Spring 容器. 连接到阿里云 DashScope 的云端知识库“健身大师”，让你的聊天模型能用云端文档做 RAG 检索增强。

```xml
/**
 * Rag云知识库Advisor配置类
 * 构造一个 RetrievalAugmentationAdvisor对象，用于增强检索能力
 */
@Configuration
public class AppRagCloudAdvisorConfig {

    @Value("${spring.ai.dashscope.api-key}")
    private String dashscopeApiKey;

    @Bean
    Advisor appRagCloudAdvisor() {
        return RetrievalAugmentationAdvisor.builder()
                .documentRetriever(
                        new DashScopeDocumentRetriever(
                                new DashScopeApi(dashscopeApiKey),
                                DashScopeDocumentRetrieverOptions.builder()
                                        .withIndexName("健身大师")
                                        .build()
                        )
                )
                .build();
    }

}
```

流程介绍: 1. 读取API KEY

          2. 创建DocumentRetriever 

                   作用:  建立和DashScope知识库的链接, 指定使用的索引名, 以后所有检索都会在这个云端知识库里查.

          3. 构建Advisor

                 把文档检索器封装成一个 `Advisor`,以后在 `ChatClient` 里用这个 Advisor，就能自动做 **RAG 检索增强**（先查知识库，再回答问题）。

```
return RetrievalAugmentationAdvisor.builder()
        .documentRetriever(...)
        .build();
```

2. 修改 FitnessApp 类，新增 doChatWithRagCloud 方法：.advisors(appRagCloudAdvisor)
   
   1. 新增测试方法: doChatWithRagCloud：注意:`chatResponse` 是 **ChatClient 调用大模型后的完整响应对象**。`chatResponse` 其实是一整包“AI 返回结果 + 附加信息”。
      真正要展示给用户的，其实就是 AI 最终的回答文本，而不是一大包复杂对象,
      所以 return chatResponse.getResult().getOutput().getText();
           拿到模型返回的主要结果（有可能包含多个）
           从结果里提取“输出对象”

                  提取最终的回答字符串



        梳理混乱的知识点: 为什么有的配置写在构造函数里，有的写在调用时.       

            **构造函数里配置的东西**

                    **固定的、所有请求都需要的规则**：

                        **系统提示词**：比如“你是健身专家”——这个不可能每次请求都重新写。.defaultSystem()

                        **记忆功能**：每次聊天都要有对话记忆，否则没上下文。 .defaultAdvisors()

                        **ChatModel**：就是底层大模型（比如 `qwen-turbo`），不会每次都换。 .builder(chatmodel)



           **doChat 方法里配置的东西**

                    **每次调用时才知道的临时信息**：

                    **用户输入 message**：每次都不同，必须在这里传。  .user(message)

                    **会话 ID chatId**：决定取哪段历史记忆，也得在请求时才能确定。

                        .advisors(

                                advisoSpec - > advisorSpec.param(AbstractChatMemoryAdvisor

                                                                                               .CHAT_MEMORY_CONVERSATION_ID_KEY, chatId)

                        )


