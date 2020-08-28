# Nginx使用

## Nginx简介

### 正向代理和反向代理

* **正向代理隐藏了真实的客户端**。
* **反向代理隐藏了真实的服务器**

## Nginx常用命令

```
nginx -s stop #暴力停止服务
nginx -s quit #优雅停止服务
ngins -s reload #重新加载配置文件
nginx -t #查看配置文件
```

## Nginx配置使用

当正常访问一个api接口时

![](E:\DistCode\TyporaLoad\Nginx.assets\微信截图_20200823142157.png)

通过nginx可以访问一个不同的域名可以达到访问此接口的作用，nginx默认端口是80，编辑配置文件，键入nginx -s reload 

配置信息

```json
    server {
        listen       80;#侦听请求链接端口
        server_name localhost;#请求链接域

        location / {#拦截侦听全部路径
                proxy_pass http://api.k780.com/;
        }

    }
```

通过这个配置，可以通过访问localhost，达到访问上面那个接口的目的，

![](E:\DistCode\TyporaLoad\Nginx.assets\微信截图_20200823145648.png)

基本的功能就是这样，但是在项目中遇到了跨域的问题，简单来说不允许非同一域的内容，通过nginx解决一下。

在一个地址下启动一个网页，通过ajax访问这个接口，看能不能访问到数据。

```html
//js代码，在8848端口启动页面
<html>
<head>
    <title>Title</title>
	<script src="js/jquery-2.0.0.min.js"></script>
</head>
<body>
	<button onclick="getData()">获取数据</button>
</body>
<script>
	function getData(){
		$.ajax({
			 url: 'http://localhost:8888/controller/get_infotest',  
			 type: 'GET',  
			 data: {"page":1},
			 dataType: "json",
			 success: function (data) { 
				 console.log(data)
			 },  
			 error: function (data) {  
				 console.log("2"+data)
			 }  
		});  
	}
</script>
</html>
```

```java
//后台代码在8888端口启动后台
@ResponseBody
@RequestMapping(value = "/get_infotest",method = RequestMethod.GET)
public Map<String,Object> get_infotest(
    @RequestParam(value = "page",required = false) Integer page) {
    /*获取数据操作*/    
    res.put("pages",pages);
    return res;
}
```

访问结果

![](E:\DistCode\TyporaLoad\Nginx.assets\微信截图_20200823170149.png)

因为浏览器的安全策略，无法访问成功，这就是跨域问题。

使用nginx解决跨域问题，nginx配置

```json
server {
listen       8877;#侦听请求链接端口
server_name localhost;#请求链接域
location / {#拦截全部路径
    add_header Access-Control-Allow-Origin 'http://127.0.0.1:8848';#允许跨域的地址
    add_header Access-Control-Allow-Headers X-Requested-With;
    add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
    proxy_pass http://localhost:8888/;
}
}
```

将Ajax的请求地址端口改成随便一个不和原强求端口相同的端口，否则无效，然后再location里配置允许跨域的地址以及代理路径，再次查看是否访问成功。

```
Ajax请求地址
url: 'http://localhost:8877/controller/get_infotest',  
```

![](E:\DistCode\TyporaLoad\Nginx.assets\微信截图_20200823171035.png)

访问成功！

