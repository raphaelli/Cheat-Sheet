# dotNet Core Web Api

## 常用服务

 - 添加XML 内容协商服务

允许接受 浏览器/Request 请求头 Accept 为 Application/xml 并返还 XML 格式数据
```cs
//添加XML 内容协商服务
        options.OutputFormatters.Add(new XmlDataContractSerializerOutputFormatter());
```

 - 时间格式
设置服务时间格式为 UTC 时区格式/yyyy-MM-dd HH:mm:ss 
```cs
.AddJsonOptions(options =>
    {
        //Set date configurations
        //options.SerializerSettings.DateTimeZoneHandling = DateTimeZoneHandling.Utc;
        options.SerializerSettings.DateFormatString = "yyyy-MM-dd HH:mm:ss";
    });
```

## 创建并返回201
```cs
[Route("{id}", Name ="GetTodo")]
public IActionResult GetTodo(int id)
{
    var todo = TodoService.Current.Todos.SingleOrDefault(x => x.Id == id);
    if (todo == null)
    {
        return NotFound();
    }

    return Ok(todo);
}

//POST 添加todo
[HttpPost]
public IActionResult Post([FromBody] TodoCreation todo)
{
    if (todo == null)
    {
        return BadRequest();
    }

    var maxId = TodoService.Current.Todos.Max(x => x.Id);
    var newTodo = new Todo
    {
        Id = ++maxId, //ID 顺位加一
        TdName = todo.TdName,
        TdTime = todo.TdTime
    };

    TodoService.Current.Todos.Add(newTodo);

    return CreatedAtRoute("GetTodo",new {id = newTodo.Id}, newTodo);
    //return new JsonResult(todo);
}
```
如上所示，HttpPost 请求 并放回一个 `CreateAtRoute`方法，带三个字段，"Route Name","参数","对象结果"。

## Data Annotations 验证特性
