NodeLocal DNS cache
https://kim-dragon.tistory.com/281
https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/

![[Pasted image 20240203152026.png]]

node local dns 안에도 coredns가 돌고 있는데 cache용도다.
coredns는 kubedns와 거의 같다.

tada 기준 
kube-dns service 밑에 coredns pod가 있음.
kube-dns service의 ip는 10.100.0.10

node local dns의 ip는 169.254.20.10 / _10.100.0.10_ <- 얘는 뭐야?
cached core dns는 10.100.0.10과 169.254.20.10에 바인딩되는 인터페이스를 만들어놓고 대기한다.
>  참고로 이 Dummy Interface는 실제로 네트워크 트래픽을 전송을하지 않고 패킷 라우팅 역할만하는 interface입니다. 결과적으로 Cache CoreDNS는 10.96.0.10와 169.254.25.10 IP 주소로 Listen 상태로 대기하며 Domain Resolve 요청을 대기

질문: 여기서 10.100.0.10 은 cluster 내에 있는 kube-dns 서비스의 ip인데.. 노드에 있는 cached core dns가 10.100.0.10 을 먹고 있다?

kube-system/kube-proxy-config를 보면 
```
mode: "iptables"
```
로 셋업돼있다.



https://nangman14.tistory.com/m/108

https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

dnslocalcache 적용 후 169.254.20.10에 dig되는거 같은데?
https://kmaster.tistory.com/111


--- 
테스트해보니 169.254.20.10 에 UDP 53 번 포트로 dns 요청은 잘 들어가서 처리되는것 같음.
tada, tada-dev , onion-dev cluster에서 수행해봄

- node local dns 8080이 health port로 등록돼있는데 어떻게 접근하는지?
	- 169.254.20.10 으로 돼있음
- 노드에 들어가서 포트조회

```
/ # netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:50051         0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:61679         0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      -
tcp        0      0 169.254.20.10:8080      0.0.0.0:*               LISTEN      -
tcp        0      0 10.100.0.10:53          0.0.0.0:*               LISTEN      -
tcp        0      0 169.254.20.10:53        0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:41175         0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -
tcp        0      0 :::8162                 :::*                    LISTEN      -
tcp        0      0 :::8163                 :::*                    LISTEN      -
tcp        0      0 :::9092                 :::*                    LISTEN      -
tcp        0      0 :::9253                 :::*                    LISTEN      -
tcp        0      0 :::9353                 :::*                    LISTEN      -
tcp        0      0 :::10249                :::*                    LISTEN      -
tcp        0      0 :::10250                :::*                    LISTEN      -
tcp        0      0 :::9100                 :::*                    LISTEN      -
tcp        0      0 :::61678                :::*                    LISTEN      -
tcp        0      0 :::111                  :::*                    LISTEN      -
tcp        0      0 :::61680                :::*                    LISTEN      -
tcp        0      0 :::10256                :::*                    LISTEN      -
tcp        0      0 :::22                   :::*                    LISTEN      -
```


onion-dev에서 busybox 띄워놓고 
```
curl 169.254.20.10:8080/health
```
해보니까 OK 떨어지고, battery service pod에서도 되는것 확인했음.

battery service의 /etc/resolv.conf 상태
```
root@battery-service-deploy-7b6b9bb749-ngjhl:/# cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local ap-southeast-1.compute.internal
nameserver 169.254.20.10
nameserver 10.100.0.10
options rotate attempts:3 timeout:1
```

tada-dev에서  busybox 띄워놓으며는
```
wget 169.254.20.10:8080/health 
```
가 안됨 
해당 busybox가 떠있는 node에서 검사해보니 해당 주소로 health check이 됨
