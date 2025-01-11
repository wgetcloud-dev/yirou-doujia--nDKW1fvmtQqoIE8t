[合集 \- .NET Core(16\)](https://github.com)[1\.多个 .NET Core SDK 版本之间进行切换 global.json2024\-03\-12](https://github.com/chenyishi/p/18066796)[2\.HttpClientHandler VS SocketsHttpHandler2024\-03\-10](https://github.com/chenyishi/p/18064531)[3\..Net Core中使用DiagnosticSource进行日志记录2024\-03\-12](https://github.com/chenyishi/p/18068309)[4\.DiagnosticSource DiagnosticListener 无侵入式分布式跟踪2024\-03\-14](https://github.com/chenyishi/p/18071178)[5\.LoggerMessageAttribute 高性能的日志记录2024\-03\-15](https://github.com/chenyishi/p/18073599)[6\..Net Core 你必须知道的source\-generators2024\-03\-16](https://github.com/chenyishi/p/18073694)[7\..NET Core使用 CancellationToken 取消API请求2024\-03\-17](https://github.com/chenyishi/p/18075600)[8\.使用 LogProperties source generator 丰富日志2024\-03\-18](https://github.com/chenyishi/p/18078355):[slower加速器官网](https://chundaotian.com)[9\..Net Core 使用 TagProvider 与 Enricher 丰富日志2024\-03\-19](https://github.com/chenyishi/p/18081945)[10\.C\# 12 拦截器 Interceptors2024\-03\-20](https://github.com/chenyishi/p/18082725)[11\.为什么推荐在 .NET 中使用 YAML 配置文件2024\-12\-23](https://github.com/chenyishi/p/18624234)[12\..NET 9 中的 多级缓存 HybridCache2024\-12\-24](https://github.com/chenyishi/p/18626831)[13\..NET 9 增强 OpenAPI 规范，不再内置swagger2024\-12\-25](https://github.com/chenyishi/p/18629271)[14\.Scoop: 开发者多环境管理利器2024\-12\-27](https://github.com/chenyishi/p/18634070)[15\.在 .NET 中使用 Tesseract 识别图片文字01\-08](https://github.com/chenyishi/p/18658890)16\.AsyncLocal的妙用01\-10收起
`AsyncLocal`是一个在.NET中用来在同步任务和异步任务中保持全局变量的工具类。


它允许你在不同线程的同一个对象中保留一个特定值，这样你可以在不同的函数和任务中访问这个值。


这是在实现异步任务中维持一致性和优雅性的一种重要手段。


#### 用法


**创建一个AsyncLocal实例：**


你可以使用`AsyncLocal`来创建一个特定类型的实例，比如：



```
AsyncLocal<string> asyncLocalString = new AsyncLocal<string>();

```

**设置值：**


你可以通过`Value`属性设置或者读取值：



```
asyncLocalString.Value = "Hello, AsyncLocal!";

```

**读取值：**


在任务中的任意位置，你都可以读取这个值：



```
Console.WriteLine(asyncLocalString.Value);

```

**实例分享与给值变更：**


在不同任务和串行任务中，`AsyncLocal` 实例的值是不一样的：



```
asyncLocalString.Value = "Main Task";
Task.Run(() =>
{
    asyncLocalString.Value = "Child Task";
    Console.WriteLine(asyncLocalString.Value); // 输出 "Child Task"
});
Console.WriteLine(asyncLocalString.Value); // 输出 "Main Task"
```

#### 场景


**数据传递：**


异步任务中，通过`AsyncLocal`进行全局数据传递而无需通过参数传递。这对于学习和复用代码很有帮助。


**上下文管理：**


在ASP.NET Core中，`AsyncLocal` 可用于管理用户请求上下文，实现不同请求之间的传递。当你需要保存一些请求相关的信息时，这是一种很有效的方法。


**百分比信息保存：**


记录不同任务和串行任务中的特定信息，说明这些任务是如何分布的。这在分析和跟踪异步操作时应对特别有用。


**系统日志和跟踪：**


`AsyncLocal` 可用于在异步任务中保存和分享日志和跟踪信息，以便在运行时采集最有益的信息，有助于问题跟踪和分析。


#### 示例：保存日志和租户信息


以下是一个使用`AsyncLocal`保存日志和租户信息的示例：



```
using System;
using System.Threading;
using System.Threading.Tasks;

public class LogContext
{
    public string StackTrace { get; set; }
    public string UserInfo { get; set; }
}

public class TenantContext
{
    public string Name { get; set; }
}

public class Program
{
    private static AsyncLocal _logContext = new AsyncLocal();
    private static AsyncLocal _tenantContext = new AsyncLocal();

    public static async Task Main(string[] args)
    {
        _logContext.Value = new LogContext { StackTrace = "Main Stack Trace", UserInfo = "User1" };
        _tenantContext.Value = new TenantContext { Name = "Tenant A" };

        Console.WriteLine($"Initial Log Context: {_logContext.Value.StackTrace}, User: {_logContext.Value.UserInfo}, Tenant: {_tenantContext.Value.Name}");

        await Task.Run(() => LogAndProcess(new LogContext { StackTrace = "Child Stack Trace", UserInfo = "User2" }, new TenantContext { Name = "Tenant B" }));

        Console.WriteLine($"After Task Log Context: {_logContext.Value.StackTrace}, User: {_logContext.Value.UserInfo}, Tenant: {_tenantContext.Value.Name}");
    }

    private static void LogAndProcess(LogContext logContext, TenantContext tenant)
    {
        _logContext.Value = logContext;
        _tenantContext.Value = tenant;

        Console.WriteLine($"In Task Log Context: {_logContext.Value.StackTrace}, User: {_logContext.Value.UserInfo}, Tenant: {_tenantContext.Value.Name}");

        // Simulate some processing
        Task.Delay(1000).Wait();

        Console.WriteLine($"After Processing Log Context: {_logContext.Value.StackTrace}, User: {_logContext.Value.UserInfo}, Tenant: {_tenantContext.Value.Name}");
    }
}

```

在这个示例中，`AsyncLocal` 用于保存日志和租户信息，确保在不同的任务上下文中正确传递和使用这些信息。


![](https://images.cnblogs.com/cnblogs_com/chenyishi/1348350/o_240408130234_wx.png)
