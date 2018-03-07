>访问博客[dotNet Core Cheat Sheet](https://ns96.com/2018/02/07/dotnet-cheat-sheet/) 带目录模式查看！

# 环境

## 下载安装
官方下载安装链接：
- [windows下安装](https://www.microsoft.com/net/download/windows)  
- [Mac OS安装](https://www.microsoft.com/net/download/macos)  
- [Linux下安装](https://www.microsoft.com/net/download/linux)  
>Linux以Ubuntu为例，推荐使用apt方式安装——[ubuntu下apt安装](https://www.microsoft.com/net/learn/get-started/linuxubuntu)


## Docker

## CI&CD

# dotnet core WebApi 

## dotNet Core WebApi 跨域
使用cors组件实现跨域

- 引入 cors组件
```shell
dotnet add package Microsoft.AspNetCore.Cors --version 2.0.1
```

- 添加 cors服务 到 `ConfigureServices()`方法
```cs
services.AddCors(options => options.AddPolicy("CorsSample",p => p.WithOrigins("http://localhost:5000").AllowAnyMethod().AllowAnyHeader()));
```

- 设定header original 到 `Configure()`方法
```cs
//配置Cors
app.UseCors("CorsSample");

```

- 修改controller的 get 方法
```cs
namespace webApiDemo1.Controllers
{
    [Route("api/[controller]")]
    public class ValuesController : Controller
    {
        // GET api/values
        [HttpGet]
        [EnableCors("CorsSample")]
        public IEnumerable<string> Get()
        {
            return new string[] { DateTime.Now.ToString() };
        }

    }
}
```