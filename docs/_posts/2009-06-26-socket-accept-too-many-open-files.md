---
layout: post
title:  Socket accept and too many open files
excerpt: "Did ever a socket `accept()` fail on you with the _Too many open files_ error? I know it happened for me so here's a short article about the things that helped me get rid of the problem."
---


###PROBLEM

Socket `accept()` fails with error _Too many open files_.

###WHAT DOES THIS MEAN

It means the application is using more file descriptors than the maximum allowable set.


###WHAT TO DO

Let's start by finding the pid of our program using the following command:
{% highlight bash%}
ps axf | grep programName
{% endhighlight %}
Example: 
{% highlight bash%}
[root@cow ~]$ ps axf | grep nags
11476 pts/0    S+     0:01  |       _ nags - current state : working
{% endhighlight %}

Continue by counting the file descriptors opened by our application `ls -l /proc/pid/fd/ | wc -l`

Example:
{% highlight bash%}
[root@cow ~]$ ls -l /proc/11476/fd/ | wc -l
7
[root@cow ~]$ 
{% endhighlight %}

Let's now use `ulimit -n` to find the maximum limit set for open files. Alternatively you can run `ulimit -a` and look for _open files_:

Example:
{% highlight bash%}
[root@cow ~]$ ulimit -n
1024
[root@cow ~]$
{% endhighlight %}

There are two possible scenarios here: either your application is actually using that many file descriptors __or__ you have a leakage somewhere produced by not properly closing file descriptors after they did their job. In order to see which of the two we're dealing with, try to count your active connections or opened files using `netstat -ap | grep pid/ | wc -l`

Example:
{% highlight bash%}
[root@cow ~]$ netstat -ap | grep 11476/ | wc -l
3
[root@cow ~]$
{% endhighlight%}

If the application is indeed using as many file descriptors as the currently maximum limit set, the solution is to increase the maximum open files limit by using `ulimit -n maxFDs`

Example:
{% highlight bash%}
[root@cow ~]$ ulimit -n 65536
{% endhighlight%}
Please note that you must do this __before__ starting your application.


If the application is not actively using that many file descriptors it means you have a leakage somewhere. Start verifying what happens to your sockets and whether you close them properly when the connection is lost or interrupted. Monitor the number of file descriptors before connecting a test client and after disconnecting it. It should be pretty obvious if there's something wrong.

---

### Useful debug commands

Here are some useful commands for debugging your application:

####List the file descriptors used by a certain application 
`ls -l /proc/<pid>/fd/`

Example:
{% highlight bash%}
[root@cow ~]$ ls -l /proc/11476/fd/
total 6
lrwx------  1 root root 64 May 28 12:29 0 -> /dev/pts/0
lrwx------  1 root root 64 May 28 12:29 1 -> /dev/pts/0
lrwx------  1 root root 64 May 28 12:29 2 -> /dev/pts/0
lrwx------  1 root root 64 May 28 12:29 3 -> socket:[230053]
lrwx------  1 root root 64 May 28 12:29 4 -> socket:[230054]
lrwx------  1 root root 64 May 28 12:29 5 -> socket:[230111]
[root@cow ~]$
{% endhighlight%}

####List the file descriptors used by a certain application 

`netstat -ape | grep pid/`
Example:
{% highlight bash%}
[root@cow ~]$ netstat -ape | grep 11476/
tcp        0      0 192.168.3.115:8888          *:*                         LISTEN      root       230053     11476/nags - curren
tcp        0      0 192.168.3.115:50651         192.168.3.110:mysql         ESTABLISHED root       230054     11476/nags - curren
tcp        0      0 192.168.3.115:50653         192.168.3.115:9000          ESTABLISHED root       230111     11476/nags - curren
[root@cow ~]$
{% endhighlight%}

####Count the sockets in TIME_WAIT on a certain port

`netstat -an | grep ':port[[:space:]]' | grep TIME_WAIT | wc -l`

Example:
{% highlight bash%}
[root@cow ~]$ netstat -an | grep ':80[[:space:]]' | grep TIME_WAIT | wc -l
54
[root@cow ~]$
{% endhighlight%}

If you happen to have a large number of sockets in `TIME_WAIT`, it may be a sign of not properly closing the sockets.

---

###Useful links


* [http://linuxgazette.net/115/nirendra.html](http://linuxgazette.net/115/nirendra.html)
* [http://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/x4733.html](http://tldp.org/LDP/solrhe/Securing-Optimizing-Linux-RH-Edition-v1.3/x4733.html)
* [http://www.developerweb.net/forum/showthread.php?t=2941](http://www.developerweb.net/forum/showthread.php?t=2941)
