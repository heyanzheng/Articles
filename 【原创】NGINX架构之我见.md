#【原创】NGINX架构（HTTP请求）之我见

>编写此文章的主要目的是为了更好的理解Nginx，而不是用来开发Nginx，所以比较浅显、粗糙；大神请抄近路。

生产环境下，NGINX多已多进程的方式来运行，也就是说每个worker进程以单进程的形式存在，各自平等、独立的处理来自客户端的请求。一个请求，也只能在一个woker中处理，从开始到最后的结束；一个请求不可能两个woker进程分工处理。

![nginx进程模型](http://img.hb.aicdn.com/e2c9630171e9368a0c6eb8f2038fba5cf51c22822e99e-ihDY6A_fw658)
>Nginx进程处理HTTP请求模型

NG启动后，首先读取配置文件，创建socket、读取配置、绑定IP和端口，开始Listen，根据此配置fork出N个worker进程。N的数量大多数情况下都等于CPU核数，这样就可以将每个worker绑定个某一个核上，而每个客户端请求也只在一个worker中开始、结束，所以从始至终基本不会有CPU竞争和上下文的切换，从而最低限度的节省开销。（这也是为什么httpd等服务器无法达到NG高性能的重要原因之一）

每个worker都有一个独立的连接池，大小为worker_connections；如果此数值大于系统对于listenfd的限制数nofile，那么实际的最大连接数为nofile，NG也会有警告。

每个从客户端过来的请求，先与服务器端三次握手建立起TCP连接后，NG的某一个worker根据规则（后面将介绍具体规则）竞争到accept_mutex后注册listenfd读事件，在读事件中accept该连接，得到这个连接好的socket，NG对其封装成ngx_connection_t结构体，然后开始读取请求、解析请求、处理请求、产生数据后返回客户端，最后断开连接。

其中该worker创建socket时获取的连接，来自于此worker中的free_connections；每次获取一个连接时，就从此空闲链接表中读取一个，用完后再返回给空闲链接表。

如果仅如此设置的话，很多种情况下会出现一个worker获得的accept比较多，空闲链接表很快就用完了，如果不提前做控制，当accept一个新tcp连接后，由于无法得到空闲连接、且此请求又无法转交给其他worker，最终导致此tcp得不到处理，最终超时终止；而此时可能其他worker有很多空余连接却没有机会处理；很明显，这种情况是不公平的，也是不应该存在的。

NG使用ngx_accept_disabled的变量来控制一个worker是否能参与竞争accept_mutex锁，使得每个worker工作量大体相当，这也是上文提到的竞争规则。

	ngx_accept_disabled = ngx_cycle->connection_n / 8 - ngx_cycle->free_connection_n;
即当剩余连接数小于总连接数的八分之一时，ngx_accept_disabled的值才大于0，而且剩余连接越少，值越大；

	if (ngx_accept_disabled > 0) {
		ngx_accept_disabled--;
	} else {
		if (ngx_trylock_accept_mutex(cycle) == NGX_ERROR) {
			return;
		}
		if (ngx_accept_mutex_held) {
			flags |= NGX_POST_EVENTS;
		} else {
			if (timer == NGX_TIMER_INFINITE || timer > ngx_accept_mutex_delay) {
				timer = ngx_accept_mutex_delay;
			}
		}
	}

从上面可以看出，当ngx_accept_disabled大于0时，不会去尝试获取accept_mutex，而是将值减少1，直到小于0；值越大，能够尝试获取锁需要的时间也越长，从而等于让出连接机会给其他的worker。不去accept，自己的连接也就控制了下来，其他worker的连接池也就会得到利用，NG就控制了worker间连接的平衡了。

对于一个NG来说，如果是对本地资源的HTTP请求，能够建立的最大连接数，应该是worker_connections * worker_processes，而如果是作为HTTP反向代理，最大并发数应该是worker_connections * worker_processes / 2 ，因为作为反向代理，每个并发会建立与客户端的连接和与后端服务的连接，占用两个连接资源。

NG对HTTP请求都封装于ngx_http_request_t中，支持keep-alive和pipe特性，但nnginx对于pipeline中的多个请求并不是并行处理的，依然是一个请求接一个请求的处理，只是在处理第一个请求的同时，客户端就可以发送第二个请求了。


> 以上知识大部分来自于tengine团队的分享，自己理解后重新整理而成。此为[原文链接](http://tengine.taobao.org/book/chapter_02.html#id1)。
