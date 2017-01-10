---
layout: post
title: Say Hello to Retrofit
tags:
- 转载
- Android
categories: Android Retrofit
description: Android Retrofit 的使用
---


该文章转载自[LaterEqualsNever](http://blog.csdn.net/ghost_programmer)的[Say Hello to Retrofit](http://blog.csdn.net/ghost_programmer/article/details/52372065)

<p>之前对Android里常用的网络请求库OkHttp与Volley做了简单的学习归纳与总结，这里看这个系列中的最后一篇，来认识一下Retrofit。 <br>
Retrofit可以认为是OkHttp的“升级版”。之所以这么说，是因为其内部默认正是基于OkHttp来进行封装的。这点从Retrofit这个命名就可以看出端倪。 <br>
回顾一下OkHttp，我们会发现虽然是封装过后的库，但OkHttp的封装是比较“碎片化”的。所以如果不自己再进行封装，使用时代码就比较容易耦合。 <br>
而Retrofit作为其升级版，有一个最吸引人的特色就是：将所有的请求封装为interface，并通过“注解”来声明api的相关信息。让你爽到停不下来。</p>

<hr>

<h1 id="restful-api">RESTful API</h1>

<p>在正式开始了解Retrofit的使用之前，我们有必要先了解一个概念，即RESTful。这是因为Retrofit的初衷就是根据RESTful风格的API来进行封装的。 <br>
关于RESTful的学习，可以参考一下<a href="http://www.androidchina.net/3749.html">RESTful API 设计指南</a>此文。相信看完之后，就会对Restful有一个基本的认识和理解了。 <br>
但我们这里可以对RESTful的核心思想做一个最简练的总结，那就是：<strong>所谓”资源”，就是网络上的一个实体，或者说是网络上的一个具体信息。那么我们访问API的本质就是与网络上的某个资源进行互动而已。</strong>那么，因为我们实质是访问资源，所以RESTful设计思想的提出者Fielding认为： <br>
URI当中不应当出现<strong>动词</strong>，因为”<strong>资源</strong>“表示一种<strong>实体</strong>，所以应该用<strong>名词</strong>表示，而动词则应该放在HTTP协议当中。那么举个最简单的例子：</p>

<blockquote>
  <ul>
  <li>xxx.com/api/createUser</li>
  <li>xxx.com/api/getUser</li>
  <li>xxx.com/api/updateUser</li>
  <li>xxx.com/api/deleteUser</li>
  </ul>
</blockquote>

<p>这样的API风格我们应该很熟悉，但如果要遵循RESTful的设计思想，那么它们就应该变为类似下面这样：</p>

<blockquote>
  <ul>
  <li>[POST]xxx.com/api/User</li>
  <li>[GET]  xxx.com/api/User</li>
  <li>[PUT]xxx.com/api/User</li>
  <li>[DELETE]xxx.com/api/User</li>
  </ul>
</blockquote>

<p>也就是说：因为这四个API都是访问服务器的USER表，所以在RESTful里URL是相同的，而是通过HTTP不同的RequestMethod来区分增删改查的行为。 <br>
而有的时候，如果某个API的行为不好用请求方法描述呢？比如说，A向B转账500元。那么，可能会出现如下设计：</p>

<blockquote>
  <p>POST /accounts/1/transfer/500/to/2</p>
</blockquote>

<p>在RESTful的理念里，如果某些动作是HTTP动词表示不了的，你就应该把动作做成一种资源。可以像下面这样使用它：</p>

<blockquote>
  <p>　　POST /transaction HTTP/1.1 <br>
  　　Host: 127.0.0.1 <br>
  　　 <br>
  　　from=1&amp;to=2&amp;amount=500.00</p>
</blockquote>

<p>好了，当然实际来说RESTful肯定不是就这点内容。这里我们只是了解一下RESTful最基本和核心的设计理念。</p>

<hr>

<h1 id="从官方介绍初识retrofit"><font face="黑体">从官方介绍初识</font>Retrofit</h1>

<p>当我们要去学习一样新的技术，还有什么是比官方的资料更好的了呢？所以，第一步我们打开网址<a href="http://square.github.io/retrofit/">http://square.github.io/retrofit/</a>。然后开始阅读： <br>
我们发现官方的介绍非常简单粗暴，一上来就通过一个<strong>Introduction</strong>来展示了如何使用Retrofit来进行一个最基本的请求。下面我们就逐步的分析一下： <br>
<img src="http://img.blog.csdn.net/20160829100903116" alt="这里写图片描述" title=""> <br>
从上图中，我们首先注意到了一个关键的说明信息：<strong>Retrofit会将你的HTTP API转换为Java中的interface的形式</strong>。OK，接着看： <br>
<img src="http://img.blog.csdn.net/20160829102118316" alt="这里写图片描述" title=""> <br>
这里我们读到的描述是：Retrofit类可以针对之前定义的interface生成一个具体实现。我们发现官方对此解释的很言简意赅，但更通俗的来讲的话： <br>
也就是说虽然我们之前将此次请求的API信息封装为了一个接口，但我们也知道Java中接口是不能产生对象的，这时Retrofit类就站出来扮演了这个角色。 <br>
我们可以将Retrofit类看作是一个“工厂类”的角色，我们在接口中提供了此次的“产品”的生产规格信息，而Retrofit则通过信息负责为我们生产。 <br>
<img src="http://img.blog.csdn.net/20160829103032195" alt="这里写图片描述" title=""> <br>
这里我们看到了一个重要的东西“<strong>Call</strong>”：<strong>通过之前封装的请求接口对象创建的任一的Call都可以发起一个同步（或异步）的HTTP请求到远程服务器</strong>。 <br>
之后说了一些通过注解来描述request的好处，然后这个简单的Introduction就结束了。那么，现在我们来简单总结一下，目前为止我们的感受如何。 <br>
我觉得就仅从以上简单的介绍当中我们起码有两点直观感受：那就是<strong>解耦明确</strong>；<strong>使用简单</strong>。通过注解的方式描述request让人眼前一亮。但与此同时： <br>
我们发现读完Introduction后，仅仅是这个基本的用例中，都仍然有很多小细节需要我们通过实际使用之后才能搞明白。这可能在一定程度上说明了： <br>
为什么说Retrofit的使用门槛相对于其它库来说要更高一些。不过没关系，我们自己先来写一个比官方Introduction更简单的用例，再逐步深入。</p>

<hr>

<h1 id="动手写第一个demo来碰坑"><font face="黑体">动手写第一个Demo来碰坑</font></h1>

<p>现在我们已经阅读完了官方的用例介绍，乍看之下没什么，但实际用起来说不定就得碰坑。为方便测试，仍然通过servlet在本地简单的实现服务器。 <br>
我们现在的设想可能是这样的，我只是想要写一个demo来测试一下用Retrofit来成功发起一次最基本的get请求，所以最初的doGet无比简单：</p>



<pre class="prettyprint"><code class=" hljs avrasm">    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response<span class="hljs-preprocessor">.setHeader</span>(<span class="hljs-string">"Content-type"</span>, <span class="hljs-string">"text/html;charset=UTF-8"</span>)<span class="hljs-comment">;</span>
        PrintWriter <span class="hljs-keyword">out</span> = response<span class="hljs-preprocessor">.getWriter</span>()<span class="hljs-comment">;</span>
        <span class="hljs-keyword">out</span><span class="hljs-preprocessor">.print</span>(<span class="hljs-string">"{\"describe\":\"请求成功\"}"</span>)<span class="hljs-comment">;</span>
        <span class="hljs-keyword">out</span><span class="hljs-preprocessor">.flush</span>()<span class="hljs-comment">;</span>
        <span class="hljs-keyword">out</span><span class="hljs-preprocessor">.close</span>()<span class="hljs-comment">;</span>
    }</code></pre>

<p>完成了以上代码的编写，然后我们把该servlet的URL配置一下，言简意赅的，就配置为“<strong>/api/retrofitTesting</strong>”好了。这时服务器就准备完毕了。 <br>
很显然，下面我们就可以把焦点放在Android客户端的实现上来了。为了使用Retrofit，所以我们的第一步工作自然就是在自己的项目中<strong>设置依赖</strong>：</p>

<p><img src="http://img.blog.csdn.net/20160829120839372" alt="这里写图片描述" title=""></p>

<p>配置好了依赖，接着我们就可以开始编码工作了。还记得官方用例的第一步吗？所以我们要做的显然是模仿它也把我们自己的HTTP API封装成interface。</p>



<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">public</span> <span class="hljs-keyword">interface</span> DemoService {
    @GET(<span class="hljs-string">"api/retrofitTesting"</span>)
    Call&lt;ResponseInfo&gt; testHttpGet();
}
<span class="hljs-comment">// GSON - BEAN</span>
class ResponseInfo {
    String describe;
}</code></pre>

<p>现在我们再来看这个所谓的API接口，可能就更加明确一点了。首先是通过注解@GET来声明本次请求方式为GET以及注明API-URL。 <br>
而就之后声明的方法来说：从其为Call的返回类型不难猜想多半与请求的发起有关，因为如果我们对OkHttp有所了解的话，一定就记得下面这样的代码：</p>



<pre class="prettyprint"><code class=" hljs vbscript">        OkHttpClient client = <span class="hljs-keyword">new</span> OkHttpClient();
        <span class="hljs-built_in">Request</span> <span class="hljs-built_in">request</span> = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Request</span>.Builder().url(url).build();
        <span class="hljs-built_in">Response</span> <span class="hljs-built_in">response</span> = client.newCall(<span class="hljs-built_in">request</span>).<span class="hljs-keyword">execute</span>();</code></pre>

<p>newCall方法实际就是返回一个Call类型的实例。而Retrofit中的Call接口相对于OkHttp添加了一个泛型，该泛型用于说明本次请求响应的数据解析类型。 <br>
那么这里的泛型为什么是我们自己建立的一个实体类呢？其实回忆一下之前服务器在response中返回的内容(JSON)，就不难猜想到与GSON有关。 <br>
好的，我们继续按照官方用例中接下来的步骤去完善我们的demo。封装好了interface，接下来自然就是调用了，最终的代码如下：</p>



<pre class="prettyprint"><code class=" hljs java">    <span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testRetrofitHttpGet</span>() {
        <span class="hljs-comment">// step1</span>
        Retrofit retrofit = <span class="hljs-keyword">new</span> Retrofit.Builder()
                .baseUrl(<span class="hljs-string">"http://192.168.2.100:8080/TestServer/"</span>)
                .build();
        <span class="hljs-comment">// step2</span>
        DemoService service = retrofit.create(DemoService.class);
        <span class="hljs-comment">// step3</span>
        Call&lt;ResponseInfo&gt; call = service.testHttpGet();
        <span class="hljs-comment">// step4</span>
        call.enqueue(<span class="hljs-keyword">new</span> Callback&lt;ResponseInfo&gt;() {
            <span class="hljs-annotation">@Override</span>
            <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onResponse</span>(Call&lt;ResponseInfo&gt; call, Response&lt;ResponseInfo&gt; response) {
                Log.d(TAG,response.body().describe);
            }

            <span class="hljs-annotation">@Override</span>
            <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onFailure</span>(Call&lt;ResponseInfo&gt; call, Throwable t) {
                Log.d(TAG, t.getMessage());
            }
        });
   }</code></pre>

<p>有了之前的说明并对照官方示例，现在会发现以上代码很容易理解。而我们发现官方没有给出的step4其实也很熟悉，因为它和OkHttp的使用是相同的。 <br>
这个时候看上去我们的准备工作就已经完成了，于是兴致勃勃的编译并运行demo来查看一下效果，却发现收到了如下的一个异常：</p>

<p><img src="http://img.blog.csdn.net/20160829124218208" alt="这里写图片描述" title=""></p>

<p>为什么会出现这种情况呢？蛋疼啊！别急，通过异常信息的描述，我们得知这似乎与类型的转换相关，然后带着这个疑问再去查看官方文档，于是发现：</p>

<p><img src="http://img.blog.csdn.net/20160829161341827" alt="这里写图片描述" title=""></p>

<p>从上述信息我们得知Retrofit默认只能将响应体转换为OkHttp中的ResponseBody，而我们之前为Call设置的泛型类型是自定义的类型ResponseInfo 。 <br>
将JSON格式的数据转换为Java-BEAN，很自然就会想到GSON。而Retrofit如果要执行这种转换是要依赖于另一个库的，所以我们还得在项目中配置另一个依赖：</p>

<p><img src="http://img.blog.csdn.net/20160829125602515" alt="这里写图片描述" title=""></p>

<p>之后，在构造Retrofit对象的时候加上一句代码就可以了：</p>



<pre class="prettyprint"><code class=" hljs avrasm">        Retrofit retrofit = new Retrofit<span class="hljs-preprocessor">.Builder</span>()
                <span class="hljs-preprocessor">.baseUrl</span>(<span class="hljs-string">"http://192.168.2.100:8080/TestServer/"</span>)
                <span class="hljs-preprocessor">.addConverterFactory</span>(GsonConverterFactory<span class="hljs-preprocessor">.create</span>()) // 加上这一句哦，亲
                <span class="hljs-preprocessor">.build</span>()<span class="hljs-comment">;</span></code></pre>

<p>这个时候，当我们再次运行程序就没问题了，成功的得到如下的日志打印：</p>

<p><img src="http://img.blog.csdn.net/20160829125841252" alt="这里写图片描述" title=""></p>

<p>现在，经过我们的一番摸索和折腾，关于Retrofit很基本的第一个demo就捣鼓出来了。 <br>
其实这是有意义的，因为回想一下会发现：现在我们对于Retrofit大致的使用套路，在心里已经有个一二三了。</p>

<hr>

<h1 id="回到官方文档继续学习"><font face="黑体">回到官方文档继续学习</font></h1>

<p>前面我们说到对于Retrofit的使用已经有了一个基本的认识和了解，接下来要做的自然就是深入和继续学习更多的使用细节。那么很显然，回到官方吧。</p>

<p><img src="http://img.blog.csdn.net/20160829132737163" alt="这里写图片描述" title=""></p>

<p>事实上前面我们已经使用到了注解@GET，这里就是告诉我们这种注解其实对于HTTP其它常用的请求方式(GET,POST,PUT,DELETE等等)都封装了。 <br>
而对于<strong>@GET</strong>来说，我们知道HTTP-GET是可以将一些简单的参数信息直接通过URL进行上传的，所以URL又可以像下面那样使用：</p>



<pre class="prettyprint"><code class=" hljs ruby"><span class="hljs-variable">@GET</span>(<span class="hljs-string">"api/retrofitTesting?param=value"</span>)</code></pre>

<ul>
<li><h2 id="replacement-blocks与path">replacement blocks与@path</h2></li>
</ul>

<p>不知道大家注意到官方示例和我们刚才自己测试写的demo中有一个细小的差别没，就是下面这样的东西：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-comment">//官方的</span>
<span class="hljs-annotation">@GET</span>(<span class="hljs-string">"users/{user}/repos"</span>)
<span class="hljs-comment">//我们的</span>
<span class="hljs-annotation">@GET</span>(<span class="hljs-string">"api/retrofitTesting"</span>)</code></pre>

<p>我们注意到官方的API的URL中有一个”<strong>{user}</strong>“这样的东西？它的作用是什么呢？我们也能在官方文档上找到答案：</p>

<p><img src="http://img.blog.csdn.net/20160829133750305" alt="这里写图片描述" title=""></p>

<p>从上述介绍中，我们注意到一个叫做<strong>replacement blocks</strong>的东西。可以最简单的将其理解为路径替换块，用”{}”表示，与注解@path配合使用。 <br>
当然我们自己实际去使用一下能够对其有一个更加深刻的理解，所以我们将我们之前的demo修改一下，变成下面这样：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@GET</span>(<span class="hljs-string">"api/{name}"</span>)
    Call&lt;ResponseInfo&gt; testHttpGet(<span class="hljs-annotation">@Path</span>(<span class="hljs-string">"name"</span>) String apiAction);
}</code></pre>

<p>使用的方式也会有所不同，我们在调用的使用需要将对应的值传给responseInfo方法。</p>



<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service.testHttpGet(<span class="hljs-string">"retrofitTesting"</span>);</span></code></pre>

<p>这样修改过后的实际效果实际上与我们之前的demo是一样的，那么这样做的好处是什么？显然是为了解耦。以官方的例子来说： <br>
“<strong><a href="https://api.github.com/users/">https://api.github.com/users/</a>{user}/repos</strong>”中的{user}就是为了针对不同的github用户解耦。因为这里假设代入我的github，URL就将变成： <br>
“<strong><a href="https://api.github.com/users/RawnHwang/repos">https://api.github.com/users/RawnHwang/repos</a></strong>”。而github的用户千千万万，如果使用我们之前的方式代码就会如下：</p>



<pre class="prettyprint"><code class=" hljs ruby"><span class="hljs-variable">@GET</span>(<span class="hljs-string">"users/RawnHwang/repos"</span>)</code></pre>

<p>这二者的优劣一目了然，我们肯定不会想要为了获取不同的user的repos去写N多个套路完全相同的API - interface吧。</p>

<ul>
<li><h2 id="query">@Query</h2></li>
</ul>

<p>前面我们讲到：对于@GET来说，参数信息是可以直接放在url中上传的。那么你马上就反应过来了，这一样也存在严重的耦合！于是，就有了@query。</p>

<p><img src="http://img.blog.csdn.net/20160829135851300" alt="这里写图片描述" title=""></p>

<p>同样，为了便于理解，我们仍然自己来实际的使用一下。就用我们之前的举到的例子好了，这里我们用@query来替换我们之前说到的如下代码：</p>

<pre class="prettyprint"><code class=" hljs ruby"><span class="hljs-variable">@GET</span>(<span class="hljs-string">"api/retrofitTesting?param=value"</span>)</code></pre>

<p>替换为：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@GET</span>(<span class="hljs-string">"api/retrofitTesting"</span>)
    Call&lt;ResponseInfo&gt; testHttpGet(<span class="hljs-annotation">@Query</span>(<span class="hljs-string">"param"</span>) String param);
}</code></pre>

<p>调用的时候改为：</p>



<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service.testHttpGet(<span class="hljs-string">"value"</span>);</span></code></pre>

<p>然后就可以在服务器查看是否成功接收到参数了：</p>



<pre class="prettyprint"><code class=" hljs cs">    <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doGet</span>(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        System.<span class="hljs-keyword">out</span>.println(request.getParameter(<span class="hljs-string">"param"</span>));
        }</code></pre>

<ul>
<li><h2 id="querymap">@QueryMap</h2></li>
</ul>

<p>聪明的你现在肯定还不满足，因为可能有这样的疑问：假设我要在参数中上传10个参数呢？这意味着我要在方法中声明10个@Query参数？当然不是！</p>

<p><img src="http://img.blog.csdn.net/20160829140827063" alt="这里写图片描述" title=""></p>

<p>我们看到了，Retrofit也考虑到了这点，所以针对于复杂的参数上传，为我们准备了@QueryMap。现在来修改我们自己的demo：</p>



<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-keyword">public</span> <span class="hljs-keyword">interface</span> DemoService {
    @GET(<span class="hljs-string">"api/retrofitTesting"</span>)
    Call&lt;ResponseInfo&gt; testHttpGet(@QueryMap Map&lt;String,String&gt; <span class="hljs-keyword">params</span>);
}</code></pre>

<p>调用的时候自然也发生了变化：</p>



<pre class="prettyprint"><code class=" hljs lasso">        <span class="hljs-built_in">Map</span><span class="hljs-subst">&lt;</span><span class="hljs-built_in">String</span>,<span class="hljs-built_in">String</span><span class="hljs-subst">&gt;</span> <span class="hljs-keyword">params</span> <span class="hljs-subst">=</span><span class="hljs-literal">new</span> HashMap<span class="hljs-subst">&lt;&gt;</span>();
        <span class="hljs-keyword">params</span><span class="hljs-built_in">.</span>put(<span class="hljs-string">"param1"</span>,<span class="hljs-string">"value1"</span>);
        <span class="hljs-keyword">params</span><span class="hljs-built_in">.</span>put(<span class="hljs-string">"param2"</span>,<span class="hljs-string">"value2"</span>);
        Call<span class="hljs-subst">&lt;</span>ResponseInfo<span class="hljs-subst">&gt;</span> call <span class="hljs-subst">=</span> service<span class="hljs-built_in">.</span>testHttpGet(<span class="hljs-keyword">params</span>);</code></pre>

<ul>
<li><h2 id="post">@POST</h2></li>
</ul>

<p>有了之前的基础，现在我们免不了要捣鼓一下POST。相信有了@GET的使用经验，如果只是想发发简单的POST请求是没有多大的难度，我们关注下POST的BODY，即请求体的使用技巧。</p>

<p><img src="http://img.blog.csdn.net/20160829143713448" alt="这里写图片描述" title=""></p>

<p>通过官方文档，我们发现出现了一个新的东西叫做“@Body”，顾名思义它就是用来封装请求体的。而同时通过后面的“User”参数类型，我们不难推断出：使用@Body时，是通过实体对象的形式来进行封装的。那么闲话少说，我们当然仍旧是自己动手来试一下： <br>
首先，我们编写一下servlet的doPost方法，假设我们这里提供一个新建User的API。完成后为该POST-request配置一个新的API URL，我们这里配置为：“<strong>/api/users</strong>”。然后，在Android端编写一个新的interface，大致如下：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/users"</span>)
    Call&lt;ResponseInfo&gt; uploadNewUser(<span class="hljs-annotation">@Body</span> User user);
}

class User{
    <span class="hljs-keyword">private</span> String name;
    <span class="hljs-keyword">private</span> String gender;
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> age;

    <span class="hljs-keyword">public</span> <span class="hljs-title">User</span>(String name, String gender, <span class="hljs-keyword">int</span> age) {
        <span class="hljs-keyword">this</span>.name = name;
        <span class="hljs-keyword">this</span>.gender = gender;
        <span class="hljs-keyword">this</span>.age = age;
    }
}

class ResponseInfo{
    String describe;
}</code></pre>

<p>这样其实就搞定了，是不是很简单。然后我们通过如下代码去调用测试就行了：</p>



<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service.uploadNewUser(new <span class="hljs-keyword">User</span>(<span class="hljs-string">"tsr"</span>,<span class="hljs-string">"male"</span>,<span class="hljs-number">24</span>));</span></code></pre>

<p>这里唯一需要说明的就是，通过@BODY这种方式封装请求体，Retrofit是通过JSON的形式来封装数据的。我们可以在服务器读取流中的数据打印：</p>



<pre class="prettyprint"><code class=" hljs json">{"<span class="hljs-attribute">name</span>":<span class="hljs-value"><span class="hljs-string">"tsr"</span></span>,"<span class="hljs-attribute">gender</span>":<span class="hljs-value"><span class="hljs-string">"male"</span></span>,"<span class="hljs-attribute">age</span>":<span class="hljs-value"><span class="hljs-number">24</span></span>}</code></pre>

<p>我单独额外说明一下这个的初衷是因为：这种情况以servlet来说，通过如下的形式是读取不到的对应的参数值的（返回null），需要自己解析。</p>



<pre class="prettyprint"><code class=" hljs avrasm">System<span class="hljs-preprocessor">.out</span><span class="hljs-preprocessor">.println</span>(request<span class="hljs-preprocessor">.getParameter</span>(<span class="hljs-string">"username"</span>))<span class="hljs-comment">;//输出为null</span></code></pre>

<ul>
<li><h2 id="formurlencoded">@FormUrlEncoded</h2></li>
</ul>

<p>这时有的朋友就说了，那我要是就想要通过request.getParameter的方式直接读取参数信息呢？没关系，也是可以的，使用@FormUrlEncoded搞定。</p>

<p><img src="http://img.blog.csdn.net/20160829150604155" alt="这里写图片描述" title=""></p>

<p>其实通过这个注解的命名，我们就很容易联系到HTTP Content-Type中的application/x-www-form-urlencoded：而其实在服务器打印Content-Type的话，会发现的确如此。也就是说，其实使用该注解过后，正是通过表单形式来上传参数的。</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@FormUrlEncoded</span>
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/users"</span>)
    Call&lt;ResponseInfo&gt; uploadNewUser(<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"username"</span>) String username,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"gender"</span>) String male,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"age"</span>) <span class="hljs-keyword">int</span> age);
}</code></pre>

<p>然后是依旧是调用，这时服务器就可以通过request.getParameter直接读取参数了：</p>



<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service.uploadNewUser(<span class="hljs-string">"tsr"</span>,<span class="hljs-string">"male"</span>,<span class="hljs-number">24</span>);</span></code></pre>

<p>当然，这个时候难免又会想起那个老梗：要写这么多的@Field参数？当然不是，也有@FieldMap供我们使用，使用方法参照@QueryMap</p>

<ul>
<li><h2 id="headers与header">@Headers与@Header</h2></li>
</ul>

<p>使用了@FormUrlEncoded之后，不知道你有没有好奇一下，假设我们的参数中含有中文信息，会不会出现乱码？让我们来验证一下：</p>



<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service.uploadNewUser(<span class="hljs-string">"张德帅"</span>,<span class="hljs-string">"male"</span>,<span class="hljs-number">24</span>);</span></code></pre>

<p>这里上传的username信息是中文，而在服务器读取后进行打印，其输出的是“?????·???”。没错，确实出现乱码了。 <br>
这个时候我们应该如何去解决呢？当然可以通过URLEncode对数据进行指定编码，然后服务器再进行对应的解码读取：</p>



<pre class="prettyprint"><code class=" hljs avrasm">        String name =  URLEncoder<span class="hljs-preprocessor">.encode</span>(<span class="hljs-string">"张德帅"</span>,<span class="hljs-string">"UTF-8"</span>)<span class="hljs-comment">;</span>
        <span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service<span class="hljs-preprocessor">.uploadNewUser</span>(name,<span class="hljs-string">"male"</span>,<span class="hljs-number">24</span>)<span class="hljs-comment">;</span></code></pre>

<p>但如果了解一点HTTP协议的使用，我们知道还有另一种解决方式：在Request-Header中设置charset信息。于是，这个时候就涉及到添加请求头了：</p>

<p><img src="http://img.blog.csdn.net/20160829155418666" alt="这里写图片描述" title=""></p>

<p>关于@Headers的使用看上去非常简单。那么，接下来我们就来修改一下我们之前的代码：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@Headers</span>(<span class="hljs-string">"Content-type:application/x-www-form-urlencoded;charset=UTF-8"</span>)
    <span class="hljs-annotation">@FormUrlEncoded</span>
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/users"</span>)
    Call&lt;ResponseInfo&gt; uploadNewUser(<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"username"</span>) String username,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"gender"</span>) String male,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"age"</span>) <span class="hljs-keyword">int</span> age);
}</code></pre>

<p>通过@Headers我们在Content-type中同时指明了编码信息，再次运行程序测试，就会发现服务器正确读取到了中文的信息。</p>

<p>除了@Headers之外，还有另一个注解叫做@Header。它的不同在于是动态的来添加请求头信息，也就是说更加灵活一点。我们也可以使用一下：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-comment">// @Headers("Content-type:application/x-www-form-urlencoded;charset=UTF-8")</span>
    <span class="hljs-annotation">@FormUrlEncoded</span>
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/users"</span>)
    Call&lt;ResponseInfo&gt; uploadNewUser(<span class="hljs-annotation">@Header</span>(<span class="hljs-string">"Content-Type"</span>) String contentType,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"username"</span>) String username,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"gender"</span>) String male,<span class="hljs-annotation">@Field</span>(<span class="hljs-string">"age"</span>) <span class="hljs-keyword">int</span> age);
}
<span class="hljs-comment">// 调用</span>
Call&lt;ResponseInfo&gt; call = service.uploadNewUser(<span class="hljs-string">"application/x-www-form-urlencoded;charset=UTF-8"</span>,<span class="hljs-string">"张德帅"</span>,<span class="hljs-string">"male"</span>,<span class="hljs-number">24</span>);</code></pre>

<ul>
<li><h2 id="读取response-header">读取response header</h2></li>
</ul>

<p>通过上面的总结我们知道通过@Header可以在请求中添加header，那么我们如何去读取响应中的header呢？我们会发现官方文档并没有相关介绍。 <br>
那么显然我们就只能自己看看了，一找发现对于Retrofit2来说Response类有一个方法叫做headers()，通过它就获取了本次请求所有的响应头。 <br>
那么，我们记得在OkHttp里面，除了headers()，还有用于获取单个指定头的header()方法。我们能不能在Retrofit里使用这种方式呢？答案是可以。 <br>
我们发现Retrofit的Response还有一个方法叫做raw()，调用该方法就可以把Retrofit的Response转换为原生的OkHttp当中的Response。而现在我们就很容器实现header的读取了吧。</p>



<pre class="prettyprint"><code class=" hljs avrasm">okhttp3<span class="hljs-preprocessor">.Response</span> okResponse = response<span class="hljs-preprocessor">.raw</span>()<span class="hljs-comment">;</span>
response<span class="hljs-preprocessor">.raw</span>()<span class="hljs-preprocessor">.header</span>(<span class="hljs-string">"Cache-Control"</span>)<span class="hljs-comment">;</span>
}</code></pre>

<hr>

<h1 id="进阶技巧">进阶技巧</h1>

<ul>
<li><h2 id="multipart-与文件上传">@Multipart 与文件上传</h2></li>
</ul>

<p>在官方文档中，实际上还有一个重要的注解，那就是@Multipart。这个使用起来相对要复杂一点，所以放在这里。下面我们就来看一下这个东西的使用。</p>

<p><img src="http://img.blog.csdn.net/20160829202043957" alt="这里写图片描述" title=""></p>

<p>可以看到官方文档上对该注解使用的说明非常简单，但这个注解使用起来却不是那么简单，这就很烦人了。不过也没关系，我们由浅入深的来看一下。</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@Multipart</span>()
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/multipartTesting"</span>)
    Call&lt;ResponseInfo&gt; testMultipart(<span class="hljs-annotation">@Part</span>(<span class="hljs-string">"part1"</span>) String part1,<span class="hljs-annotation">@Part</span>(<span class="hljs-string">"part2"</span>) String part2);
}</code></pre>

<p>这里对官方示例做了简化，因为我们在@Part参数中只使用了String类型，而没有声明RequestBody类型。接着就是调用它来进行测试：</p>



<pre class="prettyprint"><code class=" hljs sql"><span class="hljs-operator"><span class="hljs-keyword">Call</span>&lt;ResponseInfo&gt; <span class="hljs-keyword">call</span> = service.testMultipart(<span class="hljs-string">"this is the part1"</span>,<span class="hljs-string">"this is the part2"</span>);</span></code></pre>

<p>为了一探究竟，我们现在在服务器打印一下Content-Type与请求体(request-body)分别是怎样的：</p>

<p><img src="http://img.blog.csdn.net/20160829203422869" alt="这里写图片描述" title=""></p>

<p>现在我们来分析一下从打印的请求体中，我们能够得到哪些信息：</p>

<ul>
<li>首先，本次请求的Content-Type是multipart/form-data; 也就是代表通过2进制形式上传多部分的数据。</li>
<li>boundary故名思议就是分割线，它是用来对上传的不同部分的数据来进行分隔的，就像上图中体现的一样。</li>
<li>通过@Part传入的参数信息都像上图中一样被组装，首先是相关的内容信息，然后间隔一个空行，之后是实际数据内容，最后接以boundary。</li>
<li>当使用@Part上传String参数信息时，我们可以看到其默认的参数类型为application/json。 <br>
（这代表如果你的服务器接不支持解析此类型的Content-Type，就需要自己修改为对应的类型）</li>
</ul>

<p>现在我们注意到一个问题，那就是multipart/form-data将会以2进制形式来上传数据信息。那么，什么时候我们才需要2进制呢？显然就是文件的上传。 <br>
OK，那么我们继续修改代码，这次我们尝试通过@Part来上传一个文件会是什么情况？</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@Multipart</span>()
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/files"</span>)
    Call&lt;ResponseInfo&gt; uploadFile(<span class="hljs-annotation">@Part</span>(<span class="hljs-string">"file"</span>) RequestBody photo);
}
<span class="hljs-comment">//调用部分</span>
File path = Environment.getExternalStorageDirectory();
File file = <span class="hljs-keyword">new</span> File(path,<span class="hljs-string">"test.jpg"</span>);
RequestBody image = RequestBody.create(MediaType.parse(<span class="hljs-string">"image/png"</span>),file);
Call&lt;ResponseInfo&gt; call = service.uploadFile(image);</code></pre>

<p>这里可以发现，当要上传的数据是文件时，就要使用到RequestBody了。再次运行程序，然后截取服务器打印的请求体的部分进行查看：</p>

<blockquote>
  <p><img src="http://img.blog.csdn.net/20160829205250982" alt="这里写图片描述" title=""></p>
</blockquote>

<p>我们其实可以很直观的看到，大体的格式与之前都是一样的，而不同之处在于：</p>

<ul>
<li>Content-Type不再是application/json，而是image/png了。</li>
<li>空行之后的数据内容不再是之前直观的文本，而是二进制的图片内容。</li>
</ul>

<p>这个时候就会出现一个问题，假设我们为了方便，打算通过一些公有的API去测试一下上传文件(图片)什么的，会发现上传不成功。这是为什么呢？ <br>
实际上如果我们完全自己动手来写服务器，完全根据之前的请求体打印信息的格式来写解析的方法，肯定是也能完成文件上传的。但问题在于： <br>
我们说到multipart/form-data是支持多部分数据同时上传的，于是就出现了一个听上去很有逼格的梗，叫做“图文混传”。那么： <br>
为了区别于同一个request-body中的文本或文件信息，所以通常实际开发来说，上传文件的时候，Content-Disposition是这样的：</p>



<pre class="prettyprint"><code class=" hljs lasso">Content<span class="hljs-attribute">-Disposition</span>: form<span class="hljs-attribute">-data</span>; name<span class="hljs-subst">=</span><span class="hljs-string">"file"</span>；filename<span class="hljs-subst">=</span><span class="hljs-string">"test.jpg"</span></code></pre>

<p>而不是我们这里看到的：</p>



<pre class="prettyprint"><code class=" hljs lasso">Content<span class="hljs-attribute">-Disposition</span>: form<span class="hljs-attribute">-data</span>; name<span class="hljs-subst">=</span><span class="hljs-string">"file"</span></code></pre>

<p>于是别人的api里读取不到这个关键的“filename”，自然也就无法完成上传了。所以如果想通过Retrofit实现文件上传，可以通过如下方式来改造一下：</p>



<pre class="prettyprint"><code class=" hljs java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DemoService</span> {</span>
    <span class="hljs-annotation">@Multipart</span>()
    <span class="hljs-annotation">@POST</span>(<span class="hljs-string">"api/files"</span>)
    Call&lt;ResponseInfo&gt; uploadFile(<span class="hljs-annotation">@Part</span>(<span class="hljs-string">"file\";filename=\"test.jpg"</span>) RequestBody photo);
}</code></pre>

<p>然后我们在servlet服务器中进行解析，完成文件的上传。这里当然就不自己去写解析body的代码了，可以使用现有的库commons-fileupload。</p>



<pre class="prettyprint"><code class=" hljs java">    <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doPost</span>(HttpServletRequest request, HttpServletResponse response)
            <span class="hljs-keyword">throws</span> ServletException, IOException {
        <span class="hljs-keyword">try</span> {
            DiskFileItemFactory factory = <span class="hljs-keyword">new</span> DiskFileItemFactory();

            ServletFileUpload upload = <span class="hljs-keyword">new</span> ServletFileUpload(factory);

            List&lt;FileItem&gt; items = upload.parseRequest(request);<span class="hljs-comment">// 得到所有的文件</span>
            Iterator&lt;FileItem&gt; i = items.iterator();
            <span class="hljs-keyword">while</span> (i.hasNext()) {
                FileItem fi = (FileItem) i.next();
                String fileName = fi.getName();
                <span class="hljs-keyword">if</span> (fileName != <span class="hljs-keyword">null</span>) {
                    File fullFile = <span class="hljs-keyword">new</span> File(fi.getName());
                    <span class="hljs-comment">// 写入到D盘</span>
                    File savedFile = <span class="hljs-keyword">new</span> File(<span class="hljs-string">"D://"</span>, fullFile.getName());
                    fi.write(savedFile);
                }
            }
        } <span class="hljs-keyword">catch</span> (Exception e) {
            e.printStackTrace();
        }
    }</code></pre>

<p>再次运行程序，从代码中可以看到，我们希望将客户端上传的图片保存电脑的D盘根目录下，打开D盘看一下吧，已经成功上传了！ <br>
<img src="http://img.blog.csdn.net/20160829210858442" alt="这里写图片描述" title=""></p>

<ul>
<li><h2 id="不同类型的converter">不同类型的Converter</h2></li>
</ul>

<p>相信我们还记得之前我们读取服务器返回的json数据时，依赖于GsonConverterFactory就成功完成了解析。那么问题来了： <br>
假设有的时候我们服务器返回的就是一段简单的文本信息呢？这叫老夫如何是好，其实这在之前就已经有了答案了：</p>

<p><img src="http://img.blog.csdn.net/20160829211849424" alt="这里写图片描述" title=""></p>

<p>我们看到除了基于GSON的converter之外，官方还提供了很多其它的Converter。眼尖的马上就看到了Scalars后面有一个东西叫做String。 <br>
于是接下来的工作就简单了，依然是配置依赖；然后将Call的泛型指定为String；最后记得将Converter改掉，像下面这样：</p>



<pre class="prettyprint"><code class=" hljs r">//<span class="hljs-keyword">...</span>
.addConverterFactory(ScalarsConverterFactory.create())</code></pre>

<p>再次运行程序，查看日志打印信息，发现成功搞定：</p>

<p><img src="http://img.blog.csdn.net/20160829212500734" alt="这里写图片描述" title=""></p>

<p>有了这个例子，对于同类型的需求我们就很容易举一反三了。但马上又想到：假设我们的数据类型官方没有提供现成的Converter呢？很简单，自己造！</p>

<p><img src="http://img.blog.csdn.net/20160829213155049" alt="这里写图片描述" title=""></p>

<p>从官方描述中，我们发现自定义Converter其实很简单。即创建一个继承自Converter.Factory的类，然后提供自己的Converter实例就行了。 <br>
那么假设，我们现在服务返回的数据格式是下面这样的。而我们的需求是希望在客户端对其解析后，将其放进一个Map里面。</p>

<blockquote>
  <p>username=张三&amp;male=男性</p>
</blockquote>

<p>根据这样的需求，我们可以像下面这样很容易的实现我们自己的Converter：</p>



<pre class="prettyprint"><code class=" hljs axapta"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomConverterFactory</span> <span class="hljs-inheritance"><span class="hljs-keyword">extends</span></span> <span class="hljs-title">Converter</span>.<span class="hljs-title">Factory</span>{</span>

    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> CustomConverterFactory create() {
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CustomConverterFactory();
    }

    @Override
    <span class="hljs-keyword">public</span> Converter&lt;ResponseBody, ?&gt; responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit) {
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CustomConverter();
    }

    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomConverter</span> <span class="hljs-inheritance"><span class="hljs-keyword">implements</span></span> <span class="hljs-title">Converter</span>&lt;<span class="hljs-title">ResponseBody</span>,<span class="hljs-title">Map</span>&lt;<span class="hljs-title">String</span>,<span class="hljs-title">String</span>&gt;&gt;{</span>

        @Override
        <span class="hljs-keyword">public</span> Map&lt;String,String&gt; convert(ResponseBody body) throws IOException {
            Map&lt;String,String&gt; map = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
            String content = body.string();
            String [] keyValues = content.split(<span class="hljs-string">"&amp;"</span>);

            <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>;i&lt;keyValues.length;i++){
                String keyValue = keyValues[i];
                <span class="hljs-keyword">int</span> postion = keyValue.indexOf(<span class="hljs-string">"="</span>);
                String key  = keyValue.substring(<span class="hljs-number">0</span>,postion);
                String value = keyValue.substring(postion+<span class="hljs-number">1</span>,keyValue.length());
                map.put(key,value);
            }
            <span class="hljs-keyword">return</span> map;
        }
    }
}</code></pre>

<p>现在，我们就可以进行调用测试了：</p>



<pre class="prettyprint"><code class=" hljs lasso">            <span class="hljs-keyword">public</span> <span class="hljs-literal">void</span> onResponse(Call<span class="hljs-subst">&lt;</span><span class="hljs-built_in">Map</span><span class="hljs-subst">&lt;</span><span class="hljs-built_in">String</span>,<span class="hljs-built_in">String</span><span class="hljs-subst">&gt;&gt;</span> call, Response<span class="hljs-subst">&lt;</span><span class="hljs-built_in">Map</span><span class="hljs-subst">&lt;</span><span class="hljs-built_in">String</span>,<span class="hljs-built_in">String</span><span class="hljs-subst">&gt;&gt;</span> response) {
                <span class="hljs-built_in">Map</span><span class="hljs-subst">&lt;</span><span class="hljs-built_in">String</span>,<span class="hljs-built_in">String</span><span class="hljs-subst">&gt;</span> <span class="hljs-built_in">map</span> <span class="hljs-subst">=</span> response<span class="hljs-built_in">.</span>body();
                <span class="hljs-built_in">Set</span><span class="hljs-subst">&lt;</span><span class="hljs-built_in">String</span><span class="hljs-subst">&gt;</span> keySet <span class="hljs-subst">=</span> <span class="hljs-built_in">map</span><span class="hljs-built_in">.</span>keySet();

                for (<span class="hljs-built_in">String</span> key : keySet){
                    <span class="hljs-keyword">Log</span><span class="hljs-built_in">.</span>d(<span class="hljs-built_in">TAG</span>,key<span class="hljs-subst">+</span><span class="hljs-string">":"</span><span class="hljs-subst">+</span><span class="hljs-built_in">map</span><span class="hljs-built_in">.</span>get(key));
                }

            }</code></pre>

<p>查看日志打印： <br>
<img src="http://img.blog.csdn.net/20160829220127153" alt="这里写图片描述" title=""></p>

<ul>
<li><h2 id="使用interceptor">使用Interceptor</h2></li>
</ul>

<p>之前在学习@Headers的用法的时候，其实官方文档中还有一个东西，我们没有总结。官方的描述是这样的：</p>

<blockquote>
  <p>Headers that need to be added to every request can be specified using an OkHttp interceptor. <br>
  简单易懂，也就是说：如果你项目里的每一个请求需要加入某个header，那么就可以使用interceptor(interceptor本身是OkHttp里的东西)</p>
</blockquote>

<p>interceptor顾名思义就是拦截器，其具体使用方法可以参照<a href="https://github.com/square/okhttp/wiki/Interceptors">https://github.com/square/okhttp/wiki/Interceptors</a>。这里我们看下官方的第一个例子：</p>

<p><img src="http://img.blog.csdn.net/20160830093145337" alt="这里写图片描述" title=""></p>

<p>这里面的其它东西相信都很熟悉，一个新的关键叫做Interceptor.chain。很明显，从命名我们可以猜想出它就是完成拦截的关键。 <br>
事实上，我们可以对以上的示例进行简化，最后其实我们需要学习的就两行代码：</p>



<pre class="prettyprint"><code class=" hljs vbscript"><span class="hljs-built_in">Request</span> <span class="hljs-built_in">request</span> = chain.<span class="hljs-built_in">request</span>();
<span class="hljs-built_in">Response</span> <span class="hljs-built_in">response</span> = chain.proceed(<span class="hljs-built_in">request</span>);</code></pre>

<p>也就是说：</p>

<blockquote>
  <ul>
  <li>通过chain的request()方法，可以返回Request对象。</li>
  <li>通过chain的proceed()方法，可以返回此次请求的响应对象。</li>
  </ul>
</blockquote>

<p>那么，以我们之前的需求来说：对所有的请求都添加某个请求头。实际就很容易实现了。</p>



<pre class="prettyprint"><code class=" hljs java">            <span class="hljs-keyword">public</span> okhttp3.Response <span class="hljs-title">intercept</span>(Chain chain) <span class="hljs-keyword">throws</span> IOException {
                Request request = chain.request();
                <span class="hljs-comment">// 重写request</span>
                Request requestOverwrite = request.newBuilder().header(<span class="hljs-string">"User-Agent"</span>,<span class="hljs-string">"Android"</span>).build();

                <span class="hljs-keyword">return</span> chain.proceed(requestOverwrite);
            }</code></pre>

<p>同理，如果我们想要在每个Response中都添加某个header值呢？做法是一样的：</p>



<pre class="prettyprint"><code class=" hljs avrasm">            @Override
            public okhttp3<span class="hljs-preprocessor">.Response</span> intercept(Chain chain) throws IOException {
                Request request = chain<span class="hljs-preprocessor">.request</span>()<span class="hljs-comment">;</span>
                okhttp3<span class="hljs-preprocessor">.Response</span> originalResponse = chain<span class="hljs-preprocessor">.proceed</span>(request)<span class="hljs-comment">;</span>

                return originalResponse<span class="hljs-preprocessor">.newBuilder</span>()<span class="hljs-preprocessor">.header</span>(<span class="hljs-string">"Cache-Control"</span>,<span class="hljs-string">"max-age=100"</span>)<span class="hljs-preprocessor">.build</span>()<span class="hljs-comment">;</span>
            }</code></pre>

<p>但是，到现在为止，我们说到的都是interceptor在OkHttp中的使用。那么对应在Retrofit呢，其实很简单，因为我们说了Retrofit就是基于OkHttp的。 <br>
那么，前面一切的步骤都可以按照在OkHttp里的使用方法按部就班。唯一额外要做的工作就是，构建好OkHttpClient对象后，记得把它设置给Retrofit。</p>



<pre class="prettyprint"><code class=" hljs avrasm">        Retrofit retrofit = new Retrofit<span class="hljs-preprocessor">.Builder</span>()
                <span class="hljs-preprocessor">.baseUrl</span>(<span class="hljs-string">"http://192.168.2.100:8080/TestServer/"</span>)
                <span class="hljs-preprocessor">.client</span>(mOkHttpClinet) // 看这里，看这里
                <span class="hljs-preprocessor">.build</span>()<span class="hljs-comment">;</span></code></pre>

<hr>

<h1 id="总结">总结</h1>

<p>到了这里，对Android中常用的网络请求库say hello系列的三篇博客总算是总结完毕了。而针对Retrofit的此文是写的最认真的一篇，一个是因为现在它的确是流行；其次是因为个人也比较感兴趣。如果您观看后有任何想法，望多多指正，给出宝贵意见。</p></div>
