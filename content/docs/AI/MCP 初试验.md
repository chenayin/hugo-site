---
title: MCP 初试验
tags: ['MCP' ,'Spring AI','AI']
---

## Where are we?（我们现在在哪？）

现在已经拥有了数据分析平台，拥有的基本的数据查询和数据分析能力。
## Where are we going?（我们要到哪⼉去？）

但是我们的分析平台想和AI进行结合，能够拥有更灵活，更简单，更智能的查询方式，能够通过自然语言去执行对应的查询和分析工作，不再依赖一板一眼的面板操作
## How can we get there?（我们如何到达那⾥？）

增加MCP服务，衔接后端服务和智能体客户端（可以是Cursor，Trae这样的，也可以是自己开发的）

## 示例

> MCP实现方式并不拘泥于某种编程语言，本文以Java实现。


### System requirements  系统要求

Java 17 或更高版本已安装。
Spring Boot 3.3.x 或更高版本

### 依赖选择

通过 SpringBoot 项目初始化之后，再额外添加以下依赖

```xml
<dependencies>
      <dependency>
          <groupId>org.springframework.ai</groupId>
          <artifactId>spring-ai-starter-mcp-server</artifactId>
      </dependency>

      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
      </dependency>
</dependencies>
```
### Best Practices 最佳实践

> 作者提示： 这里虽说是最佳实践，我以为是必须要严格执行的。后文中的属性配置，其实就是这里要求的具体实现。

1. Use a logging library that writes to stderr or files.
   使用将日志写入 stderr 或文件的日志库。
2. Ensure any configured logging library will not write to STDOUT
   确保任何配置的日志库不会将日志写入 STDOUT

### 程序属性配置

```properties
spring.main.bannerMode=off
logging.pattern.console=
```

或者习惯yaml的话
```yaml
logging:
  pattern:
    console:
spring:
  main:
    banner-mode: off

```
相关的配置全集 ： [点击查看](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html#_configuration_properties)



### 编写Service

官方文档内容如下：

>`@Service` 注解会自动将服务注册到应用程序上下文中。Spring AI `@Tool` 注解，使得创建和维护 MCP 工具变得非常容易。The auto-configuration will automatically register these tools with the MCP server.
>自动配置会自动将这些工具注册到 MCP 服务器上。



> 作者按： 看起来与平常编写sevice，并无二异。但其中暗藏玄机
>
> 1. 和前面提到的一样， 编码过程不要出现任何打印输出的内容，如果非要打印，以err的形式打印 Sysout.err.println();
> 2. @Tool 注解会将对应的方法 注册到MCP服务器上作为工具使用

```java
package com.xmic.mcp.data.analysis.service;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.xmic.mcp.data.analysis.common.Constants;
import com.xmic.mcp.data.analysis.common.R;
import com.xmic.mcp.data.analysis.common.ResponseData;
import com.xmic.mcp.data.analysis.dto.DataQueryDTO;
import com.xmic.mcp.data.analysis.dto.ItemDTO;
import com.xmic.mcp.data.analysis.utils.JsonUtils;
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Objects;

@Service
public class DataQueryService {

    @Autowired
    private ObjectMapper objectMapper;

    private static final String BASE_URL = "http://192.168.15.172/api";

    private RestClient restClient;

    public DataQueryService() {
        this.restClient = RestClient.builder()
                .baseUrl(BASE_URL)
                .defaultHeader("Accept", "*/*")
                .defaultHeader("Content-Type", "application/json")
                .defaultHeader("User-Agent", "WeatherApiClient/1.0 (zxt@xmic.com)")
                .defaultHeader("Authorization", Constants.TOKEN_NEVER_EXPIRE)
                .build();
    }


    @Tool(description = "依据工作面code和设备因子名称查询对应的历史数据，包含 最大值，平均值，和最小值")
    public String queryHistoryDataOnFaceCode(DataQueryDTO query) {

        // 判断query中 faceCode ,factorCode 不能为空
        if (Objects.isNull(query.getFaceCode()) || Objects.isNull(query.getFactorCodes())) {
            return "faceCode 和 factorCode 不能为空";
        }

        Map<String, Object> result = new HashMap<>();
        R response = restClient.post()
                .uri("/data/history/chart/figure2D")
                .body(query)
                .retrieve()
                .body(R.class);

        if (Objects.isNull(response)) {
            result.put("msg", "查询失败了");
            return JsonUtils.toJson(result);
        }

        if (Objects.equals(response.getStatus(), R.OK)) {
            ResponseData responseData = objectMapper.convertValue(response.getData(), ResponseData.class);
            Map<String, List<List<Object>>> records = responseData.getRecords();

            Map<String, Object> map = new HashMap<>();
            for (String factorCode : records.keySet()) {
                List<List<Object>> record = records.get(factorCode);
                List<ItemDTO> list = record.stream().map(row -> ItemDTO.of(row.get(0), row.get(1), row.get(2), row.get(3))).toList();
                map.put(factorCode, list);
            }
            String json = JsonUtils.toJson(map);
            result.put("data", json);
            result.put("msg", "查询成功了");
        } else if (Objects.equals(response.getStatus(), R.NG)) {
            // 请求失败
            result.put("msg", "抱歉，获取数据时出错了,请确认参数完整后重试一下吧~");
        } else if (Objects.equals(response.getStatus(), R.KA)) {
            // 请求接口时 参数异常
            result.put("msg", "你需要提供完整的参数才能查到数据哦~");
        } else {
            result.put("msg", "查询过程中出现了未知的问题,请稍后重试或联系管理员~");
        }
        return JsonUtils.toJson(result);
    }


}

```

### 启动类

> 使用 `MethodToolCallbackProvider` 工具将 `@Tools` 转换为 MCP 服务器使用的可执行回调。

```java
package com.xmic.mcp.data.analysis;

import com.xmic.mcp.data.analysis.service.DataQueryService;
import com.xmic.mcp.data.analysis.service.DeviceFactorService;
import org.springframework.ai.tool.ToolCallbackProvider;
import org.springframework.ai.tool.method.MethodToolCallbackProvider;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class McpDataAnalysisApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpDataAnalysisApplication.class, args);
    }


    @Bean
    public ToolCallbackProvider dataQueryTools(DataQueryService dataQueryService) {
        return MethodToolCallbackProvider.builder().toolObjects(dataQueryService).build();
    }
}

```

### 客户端使用

在对应的客户端位置添加对应的mcp工具配置，添加如下内容

```json
{
  "mcpServers": {
    "spring-ai-mcp-weather": {
      "command": "java",
      "args": [
        "-Dspring.ai.mcp.server.transport=STDIO",
        "-jar",
        "C:\\ABSOLUTE\\PATH\\TO\\PARENT\\FOLDER\\weather\\mcp-weather-stdio-server-0.0.1-SNAPSHOT.jar"
      ]
    }
  }
}
```





## 参考文档

[For Server Developers - Model Context Protocol](https://modelcontextprotocol.io/quickstart/server#windows)

[spring-ai-examples 示例项目](https://github.com/spring-projects/spring-ai-examples/tree/main/model-context-protocol/weather/starter-stdio-server)

[MCP Server Boot Starter :: Spring AI Reference](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html#_configuration_properties)

