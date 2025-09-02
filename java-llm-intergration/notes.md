# 1- AI 大模型接入

## 笔记

什么是 AI 大模型？  

AI 大模型是指具有超大规模参数（通常为数十亿到数万亿）的深度学习模型，通过对大规模数据的训练，能够理解、生成人类语言，处理图像、音频等多种模态数据，并展示出强大的推理和创作能力。

如何选择大模型？对大多数开发者来说，更关注的是 准确度 + 功能支持 + 性能 + 成本。

AI 大模型面试题库：[AI大模型原理和应用面试题 - 面试鸭 - 程序员求职面试刷题神器](https://www.mianshiya.com/bank/1906189461556076546)

阿里云百炼平台：[大模型服务平台百炼控制台](https://bailian.console.aliyun.com/?tab=home#/home)

阿里云大模型服务平台百炼官方文档：[什么是阿里云百炼_大模型服务平台百炼(Model Studio)-阿里云帮助中心](https://help.aliyun.com/zh/model-studio/what-is-model-studio?spm=a2c4g.11174283.0.i1)

Spring AI 官方文档：[Introduction :: Spring AI Reference](https://docs.spring.io/spring-ai/reference/index.html)

Spring AI ALibaba 官网：[Spring AI Alibaba 官网_快速构建 JAVA AI 应用](https://java2ai.com/?spm=5176.29160081.0.0.2856aa5crvJQKf)



## 新建后端项目

### 1.在IDEA中新建项目,加入web, test,Lombook,Hutool,Knife4j等依赖

JDK版本：21

web: 快速开发 RESTful API 和 Web 应用

test: 支持 JUnit 等单元测试与集成测试。

Lombok: 通过注解自动生成 Getter、Setter、构造方法等。

Hutool: 提供字符串、时间、IO、加密等常用工具类方法。

Knife4j：Swagger 的增强 UI 工具，用于生成和美化接口文档，方便调试 API。

1. 修改 pom.xml 文件

2. 新增一个HealthController类,编写接口用于测试接口文档是否正常.
   
   1. controller: 负责接收前端（浏览器 / APP / 前端页面）发过来的请求.
   
   2. @RestController: 方法返回啥就直接给浏览器看 
      @RequestMapping:规定访问路径 
      @GetMapping: 专门处理“点链接/输入网址”这种 GET 请求

3. 删除application.properties文件，新增application.yml文件并添加配置
   
   ```xml
   spring:
     application:
       name: ai-fitness-agent
     ai:
       dashscope:
         api-key: 你的apiKey
         chat:
           options:
             model: qwen-plus
   
   server:
     port: 8080
     servlet:
       context-path: /api
   
   springdoc:
     swagger-ui:
       path: /swagger-ui.html
       tags-sorter: alpha
       operations-sorter: alpha
     api-docs:
       path: /v3/api-docs
     group-configs:
       - group: 'default'
         paths-to-match: '/**'
         packages-to-scan: com.zcq.aifitnessagent.controller
   
   knife4j:
     enable: true
     setting:
       language: zh_cn
   ```

4. 启动项目,访问接口文档: http://localhost:8080/api/doc.html
   
   ### 程序调用大模型
   
    1. 新增Spring AI Alibaba依赖，添加maven仓库配置,加入仓库配置

        2.新增调用大模型的配置:    

```yaml
spring:
  ai:
    dashscope:
      api-key: 你的apiKey
      chat:
        options:
          model: qwen-plus
```

**注意：不要提交到git中**：

        3.编写测试代码: 新增测试类

           ``` 

```
@SpringBootTest
public class AiInvokeTests {

    @Resource
    private ChatModel dashscopeChatModel;

    @Test
    public void testAiInvoke() {
        String reply = dashscopeChatModel.call(new Prompt("你好，AI智能体"))
                .getResult()
                .getOutput()
                .getText();
        System.out.println("AI reply: " + reply);
    }
}
```

@Resource: 自动帮你把需要的对象找出来塞进来，不用自己 new.


