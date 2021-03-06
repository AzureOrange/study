# 2 服务发现与负载均衡

本地使用的是haproxy-marathon-bridge来实现负载均衡和服务发现

## 2.1 原理

marathon的restful api 有查看当前程序信息的接口:

`http://192.168.1.110:8080/v2/tasks`

```
demo	10001	192.168.1.113:31001	192.168.1.115:31001	192.168.1.114:31001
tomcat	10000	192.168.1.113:31000	192.168.1.115:31000	192.168.1.114:31000

```

haproxy-marathon-bridge根据该接口生成haproxy的服务发现及负载均衡。


## 2.2 选取任意一台局域网内机器

使用的ip是`192.168.1.103`

## 2.3 安装haproxy

`yum -y install haproxy`

## 2.4 安装haproxy-marathon-bridge

```
wget https://raw.githubusercontent.com/mesosphere/marathon/master/bin/haproxy-marathon-bridge

chmod +x haproxy-marathon-bridge
```

## 2.5 生成haproxy.cfg

`./haproxy-marathon-bridge 192.168.1.110:8080 > /etc/haproxy/haproxy.cfg`

生成内容：

```
global
  daemon
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
  maxconn 4096

defaults
  log            global
  retries             3
  maxconn          2000
  timeout connect  5000
  timeout client  50000
  timeout server  50000

listen stats
  bind 127.0.0.1:9090
  balance
  mode http
  stats enable
  stats auth admin:admin

listen demo-10001
  bind 0.0.0.0:10001
  mode tcp
  option tcplog
  balance leastconn
  server demo-3 192.168.1.113:31001 check
  server demo-2 192.168.1.115:31001 check
  server demo-1 192.168.1.114:31001 check

listen tomcat-10000
  bind 0.0.0.0:10000
  mode tcp
  option tcplog
  balance leastconn
  server tomcat-3 192.168.1.113:31000 check
  server tomcat-2 192.168.1.115:31000 check
  server tomcat-1 192.168.1.114:31000 check
```

## 2.5 启动haproxy

```
systemctl start haproxy
systemctl enable haproxy
```

## 2.6 访问测试
tomcat:`http://192.168.1.103:10000`

![](https://raw.githubusercontent.com/wiselyman/study/master/mesos/resources/load1.jpg)

demo:`http://192.168.1.103:10001`
![](https://raw.githubusercontent.com/wiselyman/study/master/mesos/resources/load2.jpg)