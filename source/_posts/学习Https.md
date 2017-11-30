---
title: 学习https
date: 2017-11-30 09:00:58
tags: code
---

[学习参考](http://blog.csdn.net/clh604/article/details/22179907)

##### HTTPS

由HTTP+SSL/TLS组成。服务端配置证书可以自己制作或者权威机构购买。证书传送其实就是公钥传送，里面包含颁发机构，过期时间等。

##### 初始化

浏览器将自己支持的加密规则发送给服务端

##### 客户端解析证书

服务端选一组加密算法和hash算法，把信息征收发送给客户端。客户端TLS验证公钥是否有效，比如颁发机构，过期时间，如果异常会弹出警告。如果没有异常，就生成一个随机值。用证书对改随机值进行加密。

##### 传送加密信息

服务端得到随机值，服务端和客户端用这个随机值来进行加密解密。

##### 服务端解密信息

服务端用私钥解密之后，得到随机值（私钥），然后把内容通过该值进行对称加密。

##### 传输加密后的信息

服务端用随机值（私钥对称加密）传输内容

##### 客户端解密

客户端用对称加密（随机值）进行解密



#### ssl位置

ssl在应用层和tcp层之间。通过加入ssl头部再传到下一层。



![三次握手](http://wx2.sinaimg.cn/mw690/c1b251b3gy1flzvdmhlonj20w90yrq55.jpg)



上图梳理来自[博文代码](http://blog.csdn.net/u014386474/article/details/51669098)



[证书参考](https://www.cnblogs.com/fron/p/https-20170111.html)

## 生成服务器的密匙文件casserver.keystore



> ### keytool -genkey -alias casserver -keypass cas123 -keyalg RSA -keystore casserver.keystore -validity 365



-alias指定别名为casserver；

-keyalg指定RSA算法；

-keypass指定私钥密码；

-keystore指定密钥文件名称为casserver.keystore；

-validity指定有效期为365天。

另外提示输入密匙库口令应与-keypass指定的cas123相同.您的名字与姓氏fron.com是CAS服务器使用的域名（不能是IP，也不能是localhost），其它项随意填。

## 生成服务端证书casserver.cer

> keytool -export -alias casserver -storepass cas123 -file casserver.cer -keystore casserver.keystore

-alias指定别名为casserver；

-storepass指定私钥为liuqizhi；

-file指定导出证书的文件名为casserver.cer；

-keystore指定之前生成的密钥文件的文件名。

注意：-alias和-storepass必须为生成casserver.keystore密钥文件时所指定的别名和密码，否则证书导出失败

## 导入证书文件到cacerts 密钥库文件

接下来就是把上面生成的服务器的证书casserver.cer导入到cacerts密钥库文件中(后面的客户端会用到这些)

## 服务端Tomcat配置

server.xml是tomcat的主配置文件，Server是顶级组件，代表一个Tomcat实例，可以包含多个Services其中每个Service都有自己的Engines和Connectors



```
<Connector protocol="HTTP/1.1" SSLEnabled="true" maxThreads="150"
			scheme="https" secure="true" clientAuth="false" sslProtocol="TLS"

			keystoreFile="E:\dev_soft\apache-tomcat-9.0.0.M20\casserver.keystore"

			keystorePass="cas123" port="8443" />
```

把上面的代码贴入Services标签



配置了https之后，我们希望客户端或者其他都智能通过https来请求我们的服务，就必须禁用不安全的https请求方式，或者使http重定向为https。

解决方法：

在tomcat\conf\web.xml中的</welcome-file-list>后面加上这样一段：

```
<security-constraint>
		<web-resource-collection>
			<web-resource-name>securedapp</web-resource-name>
			<url-pattern>/*</url-pattern>
		</web-resource-collection>
		<user-data-constraint>
			<transport-guarantee>CONFIDENTIAL</transport-guarantee>
		</user-data-constraint>
	</security-constraint> 
```

这样访问https://localhost:8443/ 进入tomcat主页说明成功了。

在我的项目中，tomcat的启动是根据以下代码启动的

```java
package tomcat;

import org.apache.catalina.connector.Connector;
import org.apache.catalina.startup.Tomcat;

/**
 * The Class StartMainTomcat.
 * 
 * @author nibili
 */
public class StartTomcat1 {

	/** The Constant PORT. */
	public static final int PORT = 80;

	/** The Constant CONTEXT. */
	public static final String CONTEXT = "web_api";

	/**
	 * The main method.
	 * 
	 * @param args
	 *            the arguments
	 * @throws Exception
	 *             the exception
	 */
	public static void main(String[] args) throws Exception {
		System.setProperty("catalina.base", System.getProperty("user.dir") + "/target");
		System.setProperty("log.sql.port", "80");
		Tomcat tomcat = new Tomcat();
		tomcat.setBaseDir(System.getProperty("catalina.base"));
		tomcat.setPort(PORT);
		tomcat.addWebapp(CONTEXT, System.getProperty("user.dir") + "/src/main/webapp");
		Connector connector = tomcat.getConnector();
		connector.setURIEncoding("UTF-8");
		tomcat.start();
		System.out.println("Hit Enter in console to stop server");
		if (System.in.read() != 0) {
			tomcat.stop();
			System.out.println("Server stopped");
		}
	}
}

```

也就是说，server.xml，web.xml的配置不适用这个启动方式。





















