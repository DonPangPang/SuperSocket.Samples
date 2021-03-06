# 1 使用SuperSocket 2.0在AspNetCore项目中搭建一个Socket服务器

## 1.1 引入SuperSocket 2.0

SuperSocket 2.0目前仍然处于Beta阶段, 但是功能基本可靠, 可以用于生产环境. 

可以从[Github源地址](https://github.com/kerryjiang/SuperSocket)自行fork代码进行其它修改, 如需直接引用, 在默认的Nuget服务器是没有的, 需要添加作者自己的Nuget服务器`https://www.myget.org/F/supersocket/api/v3/index.json`,  获取预览版本. 

## 1.2 在AspNetCore中搭建一个Socket服务器

使用SuperSocket 2.0快速搭建一个Socket服务器非常简单, 在框架中, 作者实现了一个`SuperSocketHostBuilder`进行构建, 我们要做的只是简单的配置一些参数和业务逻辑. 

此外, `SuperSocketHostBuilder`是可以直接嵌入到AspNetCore项目的`CreateHostBuilder`上的, 但是并不推荐这么做...

1. 配置数据包和过滤器/选择器
   + 数据包, 即我们进行Socket通信时的数据包, 可以是包对象, 也可以是`byte[]`, 如果是包对象, 则需要在过滤器中配置解码器. 
   + 过滤器, 作用是对数据包进行筛选然后截取一包数据, 可以将解码器挂载进来, 直接将二进制数据映射为对象. 
   + 选择器, 我们可以使用选择器, 来实现一个端口服务于多种协议的情况, 此时, 一个选择器则会搭配多个过滤器进行使用. 
2. 配置IP和端口
   + 配置IP和端口可以选择两种方式, 一种方式是从程序写入, 另一种是从配置文件中写入. 
   + 在本系列的介绍中, 我们大多数采用程序写入的方式, 从配置文件写入的方式, 后续会采用其它方式实现, 来扩展业务. 
3. 配置Session
   + 在程序中作者内置了`IAppSession`和`AppSession`供我们使用, 如果我们需要自定义Session, 则需要继承`AppSession`并且在程序中进行引用, 即可切换为我们定义的Session. 
   + 自定义Session时, 由于在程序中大多数提供的参数都为`IAppSession`, 所以, 需要实现SuperSocket的更多其它接口进行重写, 来维持程序运转. 
   + 自定义Session大多数时候是为了添加更多自定义属性, 作者在设计中提供了另外的方式提供我们选择：
        ```csharp
        // AppSession源码
        ```
        可以看到, 其中内置了一个字典, 我们可以将属性的`Key-Value`值直接存在字典中, 即`session["prop"] = value;`的形式. 
4. 配置SessionHandler
   + SessionHandler会在建立和断开Socket连接时触发, 用于处理连接和断开时的业务, 可以在`SuperSocketHostBuilder`中直接配置, 也可以通过重写相应的接口实现. 
   + 连接时, 会将创建好的Session传入该方法. 
   + 断开时, 会将断开的Session以及触发断开的事件传入该方法. 
5. 配置PackageHandler
   + PackageHandler会在接收到数据包时触发. 
   + 该方法自动触发时我们将获取两个参数, 一个是Session, 一个是Package. **但是要注意的是, 这里的Session是`IAppSession`类型, 并不是我们自定义的Session**. 
     + Session即为当前Socket连接, 里面附带了各种连接信息以及状态等. 
     + Package是通过过滤器后得到的数据包, 不过要注意的是, **例如：如果数据包为头尾标识符的数据包, 如果采用的是`byte[]`的形式, 得到的可能不是原包, 而是去除了包头尾标识后的数据体. 即：`7E 01 02 03 7E`, 去掉头尾的`7E`标识得到的是`01 02 03`**. 
6. Build & Run
   + 构建这里同时提供了几种方式, 推荐采用`BuildAsServer()`, 然后通过`StartAsync()`进行启用. 

此时, 一个Socket服务器就搭建完成了. 具体实现：
```csharp
// SampleSession
public class SampleSession: AppSession
{
    
}
```

```csharp
// Socket Server代码
```