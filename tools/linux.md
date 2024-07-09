#### netstat
* -a (all) 显示所有选项，默认不显示LISTEN相关。
* -t (tcp) 仅显示tcp相关选项。
* -u (udp) 仅显示udp相关选项。
* -n 拒绝显示别名，能显示数字的全部转化成数字。
* -l 仅列出有在 Listen (监听) 的服务状态。
* -p 显示建立相关链接的程序名
* -r 显示路由信息，路由表
* -e 显示扩展信息，例如uid等
* -s 按各个协议进行统计
* -c 每隔一个固定时间，执行该netstat命令。

#### ss
* -l (listening) 显示监听状态的套接字(sockets)
* -n (numeric) 不解析服务名称
* -r (resolve) 解析主机名
* -a (all) 显示所有套接字(sockets)
* -e (extended) 显示详细的套接字(sockets)信息
* -m (memory) 显示套接字(socket)的内存使用情况
* -p (processes) 显示使用套接字(socket)的进程
* -i (info) 显示 TCP内部信息
* -s (summary) 显示套接字(socket)使用概况
* -4 (ipv4) 仅显示IPv4的套接字(sockets)
* -6 (ipv6) 仅显示IPv6的套接字(sockets)
* -t (tcp) 仅显示 TCP套接字(sockets)
* -u (udp) 仅显示 UCP套接字(sockets)

#### tcp 信息
* cat /proc/net/tcp
