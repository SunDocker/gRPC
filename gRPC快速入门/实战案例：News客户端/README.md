# 实战案例：News客户端

1. 引入与服务端相同的maven依赖与插件
2. 同样在src下建立与java目录平级的proto目录，将服务端的.proto文件copy过来
3. 同样的操作，生成grpc的存根类与通信类，然后都copy过来
4. 编写客户端调用代码
   - 创建`ManagedChannel`
   - 获取远程调用存根，创建时要传入`channel`
   - 构建参数对象，构造器模式设置对应属性即可
   - 调用方法，返回类型即为proto文件中定义的类型，同样有`getter`（但好像没`setter`？）
5. `try...final`关闭`channel`

```java
public class NewsClient {
    private static final String host = "localhost";
    private static final int port = 9999;

    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder.forAddress(host, port).usePlaintext().build();
        try {
            NewsServiceGrpc.NewsServiceBlockingStub blockingStub = NewsServiceGrpc.newBlockingStub(channel);
            NewsProto.NewsRequest request = NewsProto.NewsRequest.newBuilder().setDate("20220102").build();
            NewsProto.NewsResponse response = blockingStub.list(request);
            response.getNewsList().forEach((news) -> System.out.println(news.getTitle() + ":" + news.getContent()));
        } finally {
            channel.shutdown();
        }
    }
} 
```

