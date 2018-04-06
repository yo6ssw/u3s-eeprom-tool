---
layout: post
title: Apache Benchmark apr_socket_recv error when performing a load test
tags:
    - networking
excerpt: "Apache Benchmark crashes with `apr_socket_recv: Connection reset by peer (104)` when performing a load test"
---
### Problem

Apache Benchmark breaks with `apr_socket_recv: Connection reset by peer (104)` when performing a load test. This is even more annoying since we do not get results for the requests performed before this one.

### Context
- linux host, Ubuntu 14.04, 3.11.0-17-generic kernel
- performing a load test against a `jekyll serve` ran on localhost
- using `ab` v2.3 for the load test
- both `jekyll` and `ab` are being ran on the same machine

### Sample Output

```
benny@alpha:~$ ab -c 50 -n 1000 http://localhost:8080/
This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
apr_socket_recv: Connection reset by peer (104)
Total of 982 requests completed
benny@alpha:~$ 
```

It always seems to happen at the last requests that `ab` gloriously fails. 

Checking the output `dmesg` we can observe the following:

```
[2092487.521984] TCP: TCP: Possible SYN flooding on port 8080. Sending cookies.  Check SNMP counters.
```

Google yields [following response](http://serverfault.com/questions/236927/possible-syn-flooding-on-port-80-sending-cookies):

> [http://blog.dubbelboer.com/2012/04/09/syn-cookies.html](http://blog.dubbelboer.com/2012/04/09/syn-cookies.html) has an excellent writeup on this. If the connections are genuine and expected, try tuning `net.ipv4.tcp_max_syn_backlog` and `net.core.somaxconn` kernel parameters and the backlog size passed to `listen()` call in your application.

Let's check the kernel parameters:

```
benny@alpha:~$ sudo /sbin/sysctl -a | grep net.ipv4.tcp_max_syn_backlog
net.ipv4.tcp_max_syn_backlog = 512
```
and

```
benny@alpha:~$ sudo /sbin/sysctl -a | grep net.core.somaxconn
net.core.somaxconn = 128
```

Let's see what the effect of doubling the kernel parameters has.

```
benny@alpha:~$ sudo /sbin/sysctl -w net.ipv4.tcp_max_syn_backlog=1024
net.ipv4.tcp_max_syn_backlog = 1024
benny@alpha:~$ sudo /sbin/sysctl -w net.core.somaxconn=256
net.core.somaxconn = 256
benny@alpha:~$ 
```

Restarting jekyll and rerunning load test:

```
benny@alpha:~$ ab -c 50 -n 1000 http://localhost:8080/
This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
apr_socket_recv: Connection reset by peer (104)
Total of 987 requests completed
benny@alpha:~$ 
```

Nope, still no luck it seems.

### Hitting from afar

In order to isolate the problem and see whether it is a problem of apache benchmark or my server setup, I sshed to my digitalocean vps and ran apache benchmark from there.

Then, on my server I tried to increase the number of openfiles and `net.ipv4.tcp_max_syn_backlog` to see whether there's a noticeable effect:

```
sudo bash
ulimit -n 999999
sysctl -w net.ipv4.tcp_max_syn_backlog=20000
jekyl serve --port=8080
```

Still synflood logged in syslog... Drat!

```
benny@gamma:~$ ab -c 200 -n 20000 http://samplehost.com:8080/
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking samplehost.com (be patient)
Completed 2000 requests
Completed 4000 requests
Completed 6000 requests
Completed 8000 requests
Completed 10000 requests
Completed 12000 requests
Completed 14000 requests
apr_socket_recv: Connection reset by peer (104)
Total of 15015 requests completed
```
Notice that the number of total requests has also decreased with our previous settings. Hmmm, I just increased `tcp_max_syn_backlog` to 60000 and reran test:

```
sysctl -w net.ipv4.tcp_max_syn_backlog=20000
jekyl serve --port=8080
```

It crashes even faster:

```
benny@gamma:~$ ab -c 200 -n 20000 http://samplehost.com:8080/
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking samplehost.com (be patient)
Completed 2000 requests
Completed 4000 requests
apr_socket_recv: Connection reset by peer (104)
Total of 4930 requests completed
benny@gamma:~$ 
```

Let's try the other way around and lower the max syn backlog value:

```
sysctl -w net.ipv4.tcp_max_syn_backlog=1024
jekyll serve --port=8080
```

Still no luck. still dies at around 19927 requests.
Let's try disabling `net.ipv4.tcp_syncookies` altogether and rerun the test:

```
sysctl -w net.ipv4.tcp_syncookies=0
jekyll serve --port=8080
```

That apparently did it:

```
benny@gamma:~$ ab -c 200 -n 20000 http://samplehost.com:8080/
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking samplehost.com (be patient)
Completed 2000 requests
Completed 4000 requests
Completed 6000 requests
Completed 8000 requests
Completed 10000 requests
Completed 12000 requests
Completed 14000 requests
Completed 16000 requests
Completed 18000 requests
Completed 20000 requests
Finished 20000 requests


Server Software:        WEBrick/1.3.1
Server Hostname:        samplehost.com
Server Port:            8080

Document Path:          /
Document Length:        5312 bytes

Concurrency Level:      200
Time taken for tests:   114.804 seconds
Complete requests:      20000
Failed requests:        0
Write errors:           0
Total transferred:      111120000 bytes
HTML transferred:       106240000 bytes
Requests per second:    174.21 [#/sec] (mean)
Time per request:       1148.041 [ms] (mean)
Time per request:       5.740 [ms] (mean, across all concurrent requests)
Transfer rate:          945.22 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       39  316 1992.7     47   63230
Processing:    41   58  80.5     53    8351
Waiting:       40   55  79.1     50    8253
Total:         81  374 1996.6    100   63510

Percentage of the requests served within a certain time (ms)
  50%    100
  66%    104
  75%    107
  80%    110
  90%    363
  95%   1103
  98%   3102
  99%   7107
 100%  63510 (longest request)
benny@gamma:~$ 
```
However, this time syslog yields a different message:

```
Jul  8 09:25:57 alpha kernel: [2141575.522620] TCP: TCP: Possible SYN flooding on port 8080. Dropping request.  Check SNMP counters.
```

### Resources

- [http://www.lognormal.com/blog/2012/09/27/linux-tcpip-tuning/](http://www.lognormal.com/blog/2012/09/27/linux-tcpip-tuning/)
- [http://serverfault.com/questions/236927/possible-syn-flooding-on-port-80-sending-cookies](http://serverfault.com/questions/236927/possible-syn-flooding-on-port-80-sending-cookies)
- [http://blog.dubbelboer.com/2012/04/09/syn-cookies.html](http://blog.dubbelboer.com/2012/04/09/syn-cookies.html)