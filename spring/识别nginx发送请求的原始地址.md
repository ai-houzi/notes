## 识别nginx发送请求的原始地址

* 资料一

​	在代理模式下，Tomcat 如何识别用户的直接请求（URL、IP、https还是http )？ 在透明代理下，如果不做任何配置Tomcat 认为所有的请求都是 Nginx 发出来的，这样会导致如下的错误结果：

```java
	request.getScheme()  //总是 http，而不是实际的http或https
    request.isSecure()  //总是false（因为总是http）
    request.getRemoteAddr()  //总是 nginx 请求的 IP，而不是用户的IP
    request.getRequestURL()  //总是 nginx 请求的URL 而不是用户实际请求的 URL
    response.sendRedirect( 相对url )  //总是重定向到 http 上 （因为认为当前是 http 请求）	
```

​	如果程序中把这些当实际用户请求做处理就有问题了。解决方法很简单，	只	需要分别配置一下 Nginx 和 Tomcat 就好了，而不用改程序。 配置 Nginx 的转发选项：

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP  $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto  $scheme;
```

​	配置双方的 X-Forwarded-Proto 就是为了正确地识别实际用户发出的协议是 http 还是 https。X-Forwarded-For 是为了获得实际用户的 IP。 这样以上5项测试就都变为正确的结果了，就像用户在直接访问 Tomcat 一样。

​	上面https到http解决办法对context引导还是会出问题， 比如输入https://xxxx/signet 会转向到http://xxx/signet/ , http情况下：输入http://xxxx/signet会返回一个302到http://xxxx/signet/然后正常访问。

解决办法：

```nginx
# re-write redirects to http as to https, example: /home
proxy_redirect http:// https://;

http://serverfault.com/questions/145383/proxy-https-requests-to-a-http-backend-with-nginx
https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins
http://www.cyberciti.biz/faq/linux-unix-nginx-redirect-all-http-to-https/
```

作者地址：http://blog.sina.com.cn/s/blog_56d8ea900101hlhv.html

* 资料二

​	如果你的tomcat应用需要采用ssl来加强安全性，一种做法是把tomcat配置为支持ssl，另一种做法是用nginx反向代理tomcat，然后把nginx配置为https访问，并且nginx与tomcat之间配置为普通的http协议即可。下面说的是后一种方法，同时假定我们基于spring-boot来开发应用。

一、配置nginx：

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name localhost;
 
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
 
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}
```

这里有三点需要说明：

1. nginx允许一个server同时支持http和https两种协议。这里我们分别定义了http:80和https:443两个协议和端口号。如果你不需要http:80则可删除那行。
2. nginx收到请求后将通过http协议转发给tomcat。由于nginx和tomcat在同一台机里因此nginx和tomcat之间无需使用https协议。
3. 由于对tomcat而言收到的是普通的http请求，因此当tomcat里的应用发生转向请求时将转向为http而非https，为此我们需要告诉tomcat已被https代理，方法是增加X-Forwared-Proto和X-Forwarded-Port两个HTTP头信息。

二、接着再配置tomcat。基于spring-boot开发时只需在application.properties中进行配置：

```properties
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto
server.tomcat.port-header=X-Forwarded-Port
server.use-forward-headers=true
```

​	该配置将指示tomcat从HTTP头信息中去获取协议信息（而非从HttpServletRequest中获取），同时，如果你的应用还用到了spring-security则也无需再配置。

​	此外，由于spring-boot足够自动化，你也可以把上面四行变为两行：

```properties
server.tomcat.protocol_header=x-forwarded-proto
server.use-forward-headers=true
```

​	但不能只写一行：

```properties
server.use-forward-headers=true
```

​	具体请参见<http://docs.spring.io/spring-boot/docs/1.3.0.RELEASE/reference/htmlsingle/#howto-enable-https>，其中说到：

```properties
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto
The presence of either of those properties will switch on the valve
```

 	此外，虽然我们的tomcat被nginx反向代理了，但仍可访问到其8080端口。为此可在application.properties中增加一行：

```properties
server.address=127.0.0.1
```

​	这样一来其8080端口就只能被本机访问了，其它机器访问不到。

作者地址：https://www.cnblogs.com/yang-wu/p/5107899.html