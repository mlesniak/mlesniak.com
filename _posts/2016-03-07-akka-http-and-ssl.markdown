---
layout: post
title:  Akka-HTTP and SSL connections
date:   2016-03-06 11:16:49 +0100
categories: 
---

For a rest-based project I wanted to create an [akka-http](http://doc.akka.io/docs/akka-stream-and-http-experimental/2.0.3/scala/http/) server with SSL-support. Unfortunately I found only some scattered information, e.g. on [Stackoverflow](http://www.stackoverflow.com). hence I present this very short but complete tutorial.

First, we generate a self-signed certificate with

	keytool -genkey -keyalg RSA -keystore ssl-test-keystore.jks -storepass abcdef -validity 360 -keysize 2048

which creates a file ```ssl-test-keystore.jks``` which should be put on the classpath.

We start akka-http with the following configuration

{% highlight scala %}
object Boot extends App {
  implicit val system = ActorSystem()
  implicit val timeout = Timeout(5 seconds)
  implicit val materializer = ActorMaterializer()

  val serverContext: HttpsContext = {
    val password = "abcdef".toCharArray
    val context = SSLContext.getInstance("TLS")
    val ks = KeyStore.getInstance("jks")
    ks.load(getClass.getClassLoader.getResourceAsStream("ssl-test-keystore.jks"), password)
    val keyManagerFactory = KeyManagerFactory.getInstance("SunX509")
    keyManagerFactory.init(ks, password)
    context.init(keyManagerFactory.getKeyManagers, null, new SecureRandom)
    HttpsContext(context)
  }

  // Arbitrary route definition, not import for this example...
  val routes = new RouteDefinition().route
  Http().bindAndHandle(routes, interface = "0.0.0.0", port = 8443, httpsContext = Some(serverContext))
{% endhighlight %}

We can test our server with the awesome CLI utility [httpie](https://github.com/jkbrzt/httpie) with

	$ http --verify no https://localhost:8443/api/version

	HTTP/1.1 200 OK
	Content-Length: 62
	Content-Type: application/json
	Date: Sun, 06 Mar 2016 11:11:27 GMT
	Server: akka-http/2.4.2

	{
	    "version": "1.0-SNAPSHOT (Sun Mar 06 12:11:27 CET 2016)"
	}	

