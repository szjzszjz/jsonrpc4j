### 改动部分jsonrpc4j源码  
- #### 解决服务接口注解手动添加路径问题，自动配置子路径为当前接口全类名  
- #### 解除服务端baseUrl路径后面添加"/" 的限制  

### 改动后的使用描述
- 下载或git clone 源码  
- 用IDEA编译工程（因为是gradle架构的工程，需要安装gradle,如果编译过程中遇到问题，可参考[IDEA编译Gradle报错No signature of method](https://github.com/szjzszjz/jsonrpc4j/blob/master/src/main/resources/IDEA%E7%BC%96%E8%AF%91Gradle%E6%8A%A5%E9%94%99No%20signature%20of%20method%20Possible%20solutions%20asVersionComparator.docx)）  
- Gradle ---> publishToMavenLocal (双击)  
- 当控制台出现`BUILD SUCCESSFUL`说明发布成功  
- 在项目中引入依赖:  

gradle:  
```
"com.szjz.jsonrpc4j:jsonrpc4j:1.5.3"  
```  
   maven: 
```xml
<dependency>
    <groupId>com.szjz.jsonrpc4j</groupId>
         <artifactId>jsonrpc4j</artifactId>
    <version>1.5.3</version>
</dependency>
```

- 在项目中使用  

1、添加`JsonRpcConfiguration`配置类
```java
/**
 * author:szjz
 * date:2019/6/20
 * 
 * rpc 服务端和客户端配置
 */
@Slf4j
@Configuration
public class JsonRpcConfiguration {
    
    
    /** 服务端 */
    @Bean
    public AutoJsonRpcServiceImplExporter serviceImplExporter(){
        return new AutoJsonRpcServiceImplExporter();
    }

    /** 客户端 */
    @Bean
    @ConditionalOnProperty(value = {"rpc.server.url","rpc.server.scanPackage"})//两者配置有值的时候才导出客户端
    public AutoJsonRpcClientProxyCreator clientProxyCreator(@Value("${rpc.server.url}") String url,
                                                            @Value("${rpc.server.scanPackage}") String scanPackage){
        AutoJsonRpcClientProxyCreator creator = new AutoJsonRpcClientProxyCreator();
        try {
            creator.setBaseUrl(new URL(url));  //设置服务端url
        } catch (MalformedURLException e) {
            log.error("创建rpc服务端地址错误");
            e.printStackTrace();
        }
        //api模块的服务接口所在包
        creator.setScanPackage(scanPackage); //设置扫描包
        return creator;
    }
    
}
```
2、在项目的`resources/META-INF`添加`spring.factories`文件  
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.szjz.utils.jsonrpc4j.configuration.JsonRpcConfiguration
```

3、在`application.yml`文件中添加  
```yaml
rpc:
  server:
    scanPackage: com.szjz.api  #对外服务接口所在包路径
    url: http://localhost:8081/manager  #服务端baseUrl
```
