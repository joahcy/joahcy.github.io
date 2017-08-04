---
title: SpringMVC发送DELETE和PUT请求
date: 2017-06-05
categories: SpringMVC
tags: SpringMVC

---

## 了解Restful
> Restful一种软件架构风格，设计风格而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。  

Rest 风格的 URL.

- GET（SELECT）：从服务器查询，可以在服务器通过请求的参数区分查询的方式。  
- POST（CREATE）：在服务器新建一个资源，调用insert操作。  
- PUT（UPDATE）：在服务器更新资源，调用update操作。  
- DELETE（DELETE）：从服务器删除资源，调用delete语句

以 CRUD Order为例:

- 新增: /order POST 
- 修改: /order/1 PUT update?id=1 
- 获取: /order/1 GET get?id=1 
- 删除: /order/1 DELETE delete?id=1


## 如何发送 PUT 请求和 DELETE 请求呢 ?
- 1.web.xml需要配置 HiddenHttpMethodFilter  
    ```
    <!-- 把POST 转为DELETE和PUT请求 -->
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```

- 2.需要发送 POST 请求并携带一个 name="_method" 的隐藏域, 值为 DELETE 或 PUT  
	发送PUT请求
    ```
    <form action="springmvc/testRestPut/2" method="post">
        <input type="hidden" name="_method" value="PUT"/>
        <input type="submit" value="Test Rest PUT"/>
    </form>
    ```
    发送DELETE请求
    ```
    <form action="springmvc/testRestDelete/2" method="post">
        <input type="hidden" name="_method" value="DELETE"/>
        <input type="submit" value="Test Rest DELETE"/>
    </form>
    ```

## 在Controller中添加方法,响应PUT和DELETE请求
注:用 @PathVariable 注解得到请求URL中的id.  
响应PUT请求
```
@RequestMapping(value="/springmvc/testRestPut/{id}",method=RequestMethod.PUT)
public String testRestPut(@PathVariable Integer id){
	System.out.println("test rest PUT:"+id);
	return SUCCESS;
}
```
响应DELETE请求
```
@RequestMapping(value="/springmvc/testRestDelete/{id}",method=RequestMethod.DELETE)
public String testRestDelete(@PathVariable Integer id){
	System.out.println("test rest DELETE:"+id);
	return SUCCESS;
}
```
## 启动测试项目进行请求测试