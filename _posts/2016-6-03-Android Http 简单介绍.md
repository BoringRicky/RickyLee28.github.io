---
layout: post
title: Android HTTP 简单使介绍
tags:
- Android
categories: Android Http
description: Android HTTP 简单使介绍
---

现在几乎所有的Android应用或者游戏中联网功能几乎是不可缺少的。所以在我们开发中用到网络通信的机会非常多，这里简单介绍其中的Http通信。

在Android 6.0 中彻底的将Apache的HttpClient给移除掉了。而且google提倡在Android 2.3之后的网络请求中使用HttpURLConnnection,之前的HttpClient 就不再建议使用了。所以我们这里只介绍HttpURLConnection。

####准备知识
URL（Uniform Resource Locator）统一资源定位器，它是指向网络资源的地址，有时也被俗称为网页地址（网址）。如同在网络上的门牌号，我们可以通过门牌号找到我们需要的资源。它的格式如下：

`协议类型://服务器地址（必要时需加上端口号）/路径/文件名`

1. 我们常用的协议有： http，https，ftp等。
2. 服务器地址，通常叫做域名，有时也会加端口号比如在服务器后面添加“：80”端口。
3. 路径：以“/”字元区别路径中的每一个目录名称
4. 文件名：主要是具体到某个路径下载某个文件，比如是某个html页面。

####使用HttpURLConnection请求数据
我们在Android开发中其实主要是根据给定的网址其请求参数。前面说了，我们这里使用HttpURLConnection来来请求数据。这里只介绍“GET”和“POST”请求。

GET和POST使用HTTP的标准协议，用于编码和传送变量名/变量值对参数，并且使用相关的请求语义。每个HTTP-GET和HTTP-POST都由一系列HTTP请求头组成，这些请求头定义了客户端从服务器请求了什么，而响应则是由一系列HTTP应答头和应答数据组成，如果请求成功则返回应答。

GET以使用MIME类型application/x-www-form-urlencoded的urlencoded文本的格式传递参数。Urlencoding是一种字符编码，保证被传送的参数由遵循规范的文本组成，例如一个空格的编码是"%20"。附加参数还能被认为是一个查询字符串。与HTTP-GET类似，HTTP-POST参数也是被URL编码的。然而，变量名/变量值不作为URL的一部分被传送，而是放在实际的HTTP请求消息内部被传送。


使用HttpURLConnection的一般步骤如下：

1. 准备好我们需要请求的URL
2. 使用URL.openConnection()获取到一个HttURLConnection 对象；这一步需要强制类型转换
3. 准备请求所需的一些的方法属性。例如请求头，请求方式和超时时间等等。
4. 通过HttpURLConnection.getInputStream获取到请求返回的内容
5. 读取获取到的数据
6. 解除连接

我们先把一些get跟post类似的设置抽取成一个方法：

    private HttpURLConnection openConnection(String url) throws IOException {
        //获得URL对象
        URL parseURL = new URL(url);
        //获得HttpURLConnection对象
        HttpURLConnection connection = (HttpURLConnection) parseURL.openConnection();
        //设置连接超时
        connection.setConnectTimeout(TIMEOUT_MS);
        //设置读取超时时间
        connection.setReadTimeout(TIMEOUT_MS);
        //不使用缓存
        connection.setUseCaches(false);
        //设置是否从httpUrlConnection读入，默认情况下是true;
        connection.setDoInput(true);
        //设置httpUrlConnection是否可以输出数据，默认情况下是false;
        connection.setDoOutput(true);
        return connection;
    }

#####GET 请求
下面是GET的示例：

	private String get() throws IOException {
        StringBuilder builder = new StringBuilder();
        String urlString = "https://www.baidu.com/";
        //1.准备URL
        URL url = new URL(urlString);
        //2.获取HttpURLConnection 对象
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        //3.设置请求方式和超时时间等
        connection.setConnectTimeout(5000);
        connection.setReadTimeout(5000);
        //设置请求方式为GET请求
        connection.setRequestMethod("GET");
        connection.setDoInput(true);
        //4.获取到请求返回的输入流
        InputStream inputStream = connection.getInputStream();
        //5.读取输入流里的具体数据
        byte[] buffer = new byte[1024];
        int len = 0;
        while ((len = inputStream.read(buffer)) != -1) {
            builder.append(new String(buffer, 0, len));
        }
        //6.关闭连接
        connection.disconnect();
        return builder.toString();
    }

上面的示例是使用的“GET”方式请求，返回的是一个String，我们获取到String后，可以任意处置它们了。

#####POST 请求

Android POST 请求跟GET请求有些区别，它的请求参数不是在URL里面拼成的，而是通过流的形式提交给服务器的。


首先我们把各个参数抽取出来，放到一个Map里面，比如：

	Map<String,String> parames = new HashMap<>();
    parames.put("key1","value1");
	parames.put("key2","value2");

然后我们把这个Map里的数据拼成String，然后将其装换成byte[],如下：

	 private byte[] getParamsBody(Map<String, String> params) throws UnsupportedEncodingException {
        byte[] encodeBytes = null;
        if (params == null || params.isEmpty()) {
            return encodeBytes;
        }
        StringBuilder encodedParams = new StringBuilder();
        for (Map.Entry<String, String> entry : params.entrySet()) {
            encodedParams.append(URLEncoder.encode(entry.getKey(), ENCODING));
            encodedParams.append('=');
            if (!TextUtils.isEmpty(entry.getValue())) {
                encodedParams.append(URLEncoder.encode(entry.getValue(), ENCODING));
            } else {
                encodedParams.append(URLEncoder.encode("", ENCODING));
            }
            encodedParams.append('&');
        }

        encodeBytes = encodedParams.toString().getBytes();
        return encodeBytes;
    }

这样我们在post请求的时候，就可以直接将byte[]写到流里就好，代码如下：

    public String post(String url, Map<String, String> parames) throws IOException {
        if (parames == null || parames.isEmpty()) {
            return get(url);
        }
        //获取参数的byte[]
        byte[] data = getParamsBody(parames);
        HttpURLConnection connection = openConnection(url);
        //使用POST请求
        connection.setRequestMethod("POST");
        //setDoOutput 设置为true后才能写入参数
        //设置 内容类型
        connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
        //设置 内容得长度
        connection.setRequestProperty("Content-Length", String.valueOf(data.length));
        //通过HttpURLConnection 的输出流 将POST请求的参数传递出去
        OutputStream outputStream = connection.getOutputStream();
        //将参数写入到 HttpURLConnection 的输出流里
        outputStream.write(data);
        return getContent(connection);
    }


#### GET与POST区别
上面简单介绍了一些GET与POST的使用，这两种方法虽然都可作为与服务器交互的方式，但是它们还是有一些区别的：

1. get是从服务器上获取数据，post是向服务器传送数据。
2.  在客户端，Get方式在通过URL提交数据，数据在URL中可以看到；POST方式，数据放置在IO流内提交。
3.  GET方式提交的数据最多只能有1024字节，而POST则没有此限制。
4.  在安全性方面，Get方式是直接把参数放到了URL里，我们可以看到;而Post 是在流里的提交，不容易被看到。

参考：[Http之Get/Post请求区别](http://www.cnblogs.com/wxf0701/archive/2008/08/17/1269798.html)


