导语
对于 web 服务，不管是上新，还是平时产品运营，节假日活动等，在这之前都需要评估现网压力承受能力，提前进行扩容，并做到防患于未然。所以对网站性能压力测试是必不可少的，这样才能充分了解自己部署的 web 服务 QPS。只有让服务器处在高压情况下才能真正体现出各种设置所暴露的问题。Apache 中有个自带的，名为 ab 的程序，可以对 Apache 或其它类型的服务器进行网站访问压力测试。

1. ApacheBench 命令原理
ab 命令会创建很多的并发访问线程，模拟多个访问者同时对某一 URL 地址进行访问。它的测试目标是基于 URL 的，因此，既可以用来测试 Apache 的负载压力，也可以测试 nginx、lighthttp、tomcat、IIS 等其它 Web 服务器的压力。

ab 命令对发出负载的计算机要求很低，既不会占用很高 CPU，也不会占用很多内存，但却会给目标服务器造成巨大的负载，其原理类似 CC 攻击。自己测试使用也须注意，否则一次上太多的负载，可能造成目标服务器因资源耗完，严重时甚至导致死机。


2. ApacheBench 参数说明
1) Synopsis

```bash
ab [ -A auth-username:password ] [ -b windowsize ] [ -B local-address ] [ -c concurrency ] [ -C cookie-name=value ] [ -d ] [ -e csv-file ] [ -f protocol ] [ -g gnuplot-file ] [ -h ] [ -H custom-header ] [ -i ] [ -k ] [ -l ] [ -m HTTP-method ] [ -n requests ] [ -p POST-file ] [ -P proxy-auth-username:password ] [ -q ] [ -r ] [ -s timeout ] [ -S ] [ -t timelimit ] [ -T content-type ] [ -u PUT-file ] [ -v verbosity] [ -V ] [ -w ] [ -x
-attributes ]
 [ -X proxy[:port] ] [ -y-attributes ] [ -z-attributes ] [ -Z ciphersuite ] [http[s]://]hostname[:port]/path
 ```

2) Options

```bash
-n requests Number of requests to perform
```

//在测试会话中所执行的请求个数（本次测试总共要访问页面的次数）。默认时，仅执行一个请求。

```bash
-c concurrency Number of multiple requests to make
```

//一次产生的请求个数（并发数）。默认是一次一个。

```bash
-t timelimit Seconds to max. wait for responses
```

//测试所进行的最大秒数。其内部隐含值是-n 50000。它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。

```bash
-p postfile File containing data to POST
```

//包含了需要 POST 的数据的文件，文件格式如“p1=1&p2=2”.使用方法是 -p 111.txt 。 （配合-T）

```bash
-T content-type Content-type header for POSTing
```

//POST 数据所使用的 Content-type 头信息，如 -T “application/x-www-form-urlencoded” 。 （配合-p）

```bash
-v verbosity How much troubleshooting info to print
```

//设置显示信息的详细程度 – 4 或更大值会显示头信息， 3 或更大值可以显示响应代码(404, 200 等), 2 或更大值可以显示警告和其他信息。 -V 显示版本号并退出。

```bash
-w Print out results in HTML tables
```

//以 HTML 表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。

```bash
-i Use HEAD instead of GET
```

// 执行 HEAD 请求，而不是 GET。

```bash
-x attributes String to insert as table attributes
-y attributes String to insert as tr attributes
-z attributes String to insert as td or th attributes
-C attribute Add cookie, eg. -C “c1=1234,c2=2,c3=3” (repeatable)
```

//-C cookie-name=value 对请求附加一个 Cookie:行。 其典型形式是 name=value 的一个参数对。此参数可以重复，用逗号分割。

提示：可以借助 session 实现原理传递 JSESSIONID 参数， 实现保持会话的功能，如

```bash
-C ” c1=1234,c2=2,c3=3, JSESSIONID=FF056CD16DA9D71CB131C1D56F0319F8″ 。
-H attribute Add Arbitrary header line, eg. ‘Accept-Encoding: gzip’ Inserted after all normal header lines. (repeatable)
-A attribute Add Basic WWW Authentication, the attributes
are a colon separated username and password.
-P attribute Add Basic Proxy Authentication, the attributes
are a colon separated username and password.
```

//-P proxy-auth-username:password 对一个中转代理提供 BASIC 认证信任。用户名和密码由一个:隔开，并以 base64 编码形式发送。无论服务器是否需要(即, 是否发送了 401 认证需求代码)，此字符串都会被发送。

```bash
-X proxy:port Proxyserver and port number to use
-V Print version number and exit
-k Use HTTP KeepAlive feature
-d Do not show percentiles served table.
-S Do not show confidence estimators and warnings.
-g filename Output collected data to gnuplot format file.
-e filename Output CSV file with percentages served
-h Display usage information (this message)
```

//-attributes 设置属性的字符串. 缺陷程序中有各种静态声明的固定长度的缓冲区。另外，对命令行参数、服务器的响应头和其他外部输入的解析也很简单，这可能会有不良后果。它没有完整地实现 HTTP/1.x; 仅接受某些’预想’的响应格式。 strstr(3) 的频繁使用可能会带来性能问题，即你可能是在测试 ab 而不是服务器的性能。

参数很多，常用的就两个，例如：

```bash
# ab -c 1000 -n 100 http://10.50.120.121/test/blank\
```

3. ApacheBench 用法详解
在 Linux 系统，一般安装好 Apache 后可以直接执行；

```bash
#ab -k -n 100000 -c 500 -H "Accept-Encoding: gzip, deflate, sdch" -H "User-Agent:Mozilla/5.0 (iPhone; CPU iPhone OS 10_2 like Mac OS X) AppleWebKit/602.3.12 (KHTML, like Gecko) Mobile/14C92 QQ/7.0.0.3002_GrayThree_309592_04-27_23:04 V1_IPH_SQ_7.0.0_5_HDBM_T Pixel/750 Core/UIWebView NetType/WIFI QBWebViewType/1" -C "uin=o0369491785;skey=@MC7dyF5lu;" http://10.51.170.39/sport/home?_wv=2163715
```

如果是 win 系统下，打开 cmd 命令行窗口，cd 到 apache 安装目录的 bin 目录下

上述命令的含义是：

在长连接状态下，模拟移动端，请求头带 gzip 压缩，并且带登录态，一次 500 个并发，总请求量为 100000，向指定机器 IP 和页面的 URL 发送请求
URL 也可以是域名，本地配置 HOST 指向要压测的机器 IP 也可以~

稍等片刻，执行结果及分析如下：

```bash
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.51.170.39 (be patient)
Completed 10000 requests
Completed 20000 requests
Completed 30000 requests
Completed 40000 requests
Completed 50000 requests
Completed 60000 requests
Completed 70000 requests
Completed 80000 requests
Completed 90000 requests
Completed 100000 requests
Finished 100000 requests

Server Software:                         //测试服务端类型
Server Hostname:        10.51.170.39     //测试服务器 HOST
Server Port:            80               //测试服务端端口号

Document Path:          /sport/home?_wv=2163715   //The request URI parsed from the command line string.
Document Length:        62346 bytes               //This is the size in bytes of the first successfully returned document. If the document length changes during testing, the response is considered an error.

Concurrency Level:      500                       //The number of concurrent clients used during the test
Time taken for tests:   158.969 seconds           //This is the time taken from the moment the first socket connection is created to the moment the last response is received
Complete requests:      100000                    //The number of successful responses received
Failed requests:        0                         //The number of requests that were considered a failure. 
Write errors:           0                         //he number of errors that failed during write (broken pipe).
Keep-Alive requests:    100000                    //The number of connections that resulted in Keep-Alive requests
Total transferred:      6266233588 bytes          //The total number of bytes received from the server. This number is essentially the number of bytes sent over the wire.
HTML transferred:       6236217388 bytes          //The total number of document bytes received from the server. This number excludes bytes received in HTTP headers
Requests per second:    629.05 [#/sec] (mean)     //This is the number of requests per second. This value is the result of dividing the number of requests by the total time taken
Time per request:       794.847 [ms] (mean)       //The average time spent per request. The first value is calculated with the formula concurrency * timetaken * 1000 / done while the second value is calculated with the formula timetaken * 1000 / done
Time per request:       1.590 [ms] (mean, across all concurrent requests)
Transfer rate:          38494.01 [Kbytes/sec] received            //The rate of transfer as calculated by the formula totalread / 1024 / timetaken

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   2.1      0      34
Processing:   296  793 151.5    809    2962
Waiting:      232  690 153.2    713    2724
Total:        296  793 152.1    809    2992

//整个场景中所有请求的响应情况。在场景中每个请求都有一个响应时间，其中 50％的用户响应时间小于 809 毫秒，66％的用户响应时间小于 854 毫秒，最大的响应时间小于 2992 毫秒。对于并发请求，cpu 实际上并不是同时处理的，而是按照每个请求获得的时间片逐个轮转处理的，所以基本上第一个 Time per request 时间约等于第二个 Time per request 时间乘以并发请求数。

Percentage of the requests served within a certain time (ms)
  50%    809
  66%    854
  75%    881
  80%    899
  90%    947
  95%    998
  98%   1075
  99%   1146
 100%   2992 (longest request)
 ```

4. 总结
一般我们在对 web 服务器进行压力测试时，建议使用内网的另一台或者多台服务器通过内网进行测试，这样得出的数据，准确度会高很多。如果只有单独的一台服务器，可以直接本地测试，比远程测试效果要准确。

参考文档：

https://httpd.apache.org/docs/2.4/programs/ab.html