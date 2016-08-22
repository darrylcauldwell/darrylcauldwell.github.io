---
layout: post
title:  "How To Calculate Optimal TCP Window Size For Long Distance WAN Link"
date:   2014-07-24 16:59:56 +0100
tags:
    - Networking
permalink: how-to-calculate-optimal-tcp-window-size-for-long-distance-wan-link/
---
If you have a requirement to copy large amounts of data along way around the world 
you may find that despite your link being 60Mb/s if its 5,000 miles away you only 
can transfer files at much less like 10Mb/s. The cause of this is generally the TCP 
Window Size is optimized by OS and FTP clients by default to work on networks with 
less distance and less network round trip latency.

When using TCP to transfer data the two most important factors are the TCP window 
size and the round trip latency.  If you know the TCP window size and the round trip 
latency you can calculate the maximum possible throughput of a data transfer between 
two hosts, regardless of how much bandwidth you have.

{% highlight bash %}
TCP-Window-Size-in-bits / Latency-in-seconds = Bits-per-second-throughput
{% endhighlight %}

The standard Windows TCP window is 64KB

{% highlight bash %}
64KB = 65536 Bytes = 524288 bits
{% endhighlight %}

To calculate the round trip delay time can be done in many ways, the easiest is to use 
ping.exe as this records RTD in ms.

<center><img src="/images/RTD.gif" width="50%"></center>

In the above the average latency is 38ms.  So using formula we can calculate the theoretical 
maximum.

{% highlight bash %}
524288 / 0.038 = 13797053 bp/s = 13.8 Mbp/s
{% endhighlight %}

So you might want to know what you can do to make it faster? As with any equation 
the answer is adjust part so either increase the TCP window size and/or reduce latency. 
In many ways the latency is not something we can control easily but we can easily adjust 
the TCP window size.  If we know the latency and we know the size of pipe we have we 
should be able to calculate the best TCP window size.

{% highlight bash %}
(Link speed * Latency / 8) = TCP Window size Bytes
{% endhighlight %}

So if we have a 60Mb/s link across a route 5,042 miles incurring latency of 173ms.

{% highlight bash %}
(60000000*0.173)/8 = 1297500
{% endhighlight %}

The downside to increasing the TCP window size on your server is that these larger 
packets increase the buffer size, this buffer sits in memory. Also if larger packets are 
lost, then more data is lost and therefore more needs retransmitting.
