# iptables实现DNAT

*  UDP

由于UDP不需要回复报文，所以比较简单。

```bash
iptables -t nat -I PREROUTING -d 10.185.191.142 -p udp --dport 27300 -j DNAT --to-destination 10.185.191.143:27300
```

将所有发往10.185.191.142，端口为27300，并且协议为udp的报文，转发到10.185.191.143。



如果设置之后不能成功转发报文，则还需要设置一下：

```bash
sysctl -w net.ipv4.ip_forward=1
```

即开启主机转发报文的功能，默认是关闭的。这种修改方法是修改的内存里面的值，机器重启之后就失效了。

  

* TCP

  * 将发往本机的报文重定向到127.0.0.1

    ```bash
    sysctl -w net.ipv4.conf.all.route_localnet=1
    iptables -t nat -I PREROUTING  -p tcp --dport 10000 -j DNAT --to-destination 127.0.0.1:10000
    ```

    
