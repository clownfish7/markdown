```shell
# 从仓库拉取镜像
$ sudo docker image pull delron/elasticsearch-ik:2.4.6-1.0
$ sudo docker run -d --name=elasticsearch --network=host -v /data/elasticsearch/config:/usr/share/elasticsearch/config delron/elasticsearch-ik:2.4.6-1.0

iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
```

