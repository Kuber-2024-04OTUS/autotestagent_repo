## ДЗ#13

создадим под с distroless контейнером

```console
kubectl apply -f nginx-distroless-pod.yaml
```


создадим эфемерный контейнер для отладки
```console
kubectl debug -it nginx-distroless -n default --image=busybox --target=nginx --share-processes -- sh 
```

посмотрим список процессов
```console
ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
    7 101       0:00 nginx: worker process
   26 root      0:00 sh
   36 root      0:00 ps aux

~ # ls -la /usr/local/nginx/conf
ls: /usr/local/nginx/conf: No such file or directory
```

прикрепимся другим контейнером с tcpdump'ом
```console
   kubectl debug -it nginx-distroless -n default --image=nicolaka/netshoot --target=nginx --share-processes -- sh         
Targeting container "nginx". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-6z5kl.
If you don't see a command prompt, try pressing enter.
~ # tcpdump -nn -i any -e port 80
```

создадим порт форвардинг

```console
kubectl port-forward pod/nginx-distroless 8080:80 -n default
```

запустим браузер и перейдем по адресу http://localhost:8080/
в консоли увидим результат tcpdump

```console
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
20:13:18.399908 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.56884 > 127.0.0.1.80: Flags [S], seq 1089733999, win 65495, options [mss 65495,sackOK,TS val 264548104 ecr 0,nop,wscale 7], length 0
20:13:18.399918 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.80 > 127.0.0.1.56884: Flags [S.], seq 593690778, ack 1089734000, win 65483, options [mss 65495,sackOK,TS val 264548104 ecr 264548104,nop,wscale 7], length 0
20:13:18.399926 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.56884 > 127.0.0.1.80: Flags [.], ack 1, win 512, options [nop,nop,TS val 264548104 ecr 264548104], length 0
20:13:18.415077 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 1450: 127.0.0.1.56884 > 127.0.0.1.80: Flags [P.], seq 1:1379, ack 1, win 512, options [nop,nop,TS val 264548119 ecr 264548104], length 1378: HTTP: GET / HTTP/1.1
20:13:18.415090 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.56884: Flags [.], ack 1379, win 502, options [nop,nop,TS val 264548119 ecr 264548119], length 0
20:13:18.415346 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 310: 127.0.0.1.80 > 127.0.0.1.56884: Flags [P.], seq 1:239, ack 1379, win 512, options [nop,nop,TS val 264548119 ecr 264548119], length 238: HTTP: HTTP/1.1 200 OK
20:13:18.415358 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.56884 > 127.0.0.1.80: Flags [.], ack 239, win 511, options [nop,nop,TS val 264548119 ecr 264548119], length 0
20:13:18.415375 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 684: 127.0.0.1.80 > 127.0.0.1.56884: Flags [P.], seq 239:851, ack 1379, win 512, options [nop,nop,TS val 264548119 ecr 264548119], length 612: HTTP
20:13:18.415379 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.56884 > 127.0.0.1.80: Flags [.], ack 851, win 507, options [nop,nop,TS val 264548119 ecr 264548119], length 0
20:13:18.458789 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 1404: 127.0.0.1.56884 > 127.0.0.1.80: Flags [P.], seq 1379:2711, ack 851, win 512, options [nop,nop,TS val 264548163 ecr 264548119], length 1332: HTTP: GET /favicon.ico HTTP/1.1
20:13:18.458808 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.56884: Flags [.], ack 2711, win 502, options [nop,nop,TS val 264548163 ecr 264548163], length 0
20:13:18.459014 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 380: 127.0.0.1.80 > 127.0.0.1.56884: Flags [P.], seq 851:1159, ack 2711, win 512, options [nop,nop,TS val 264548163 ecr 264548163], length 308: HTTP: HTTP/1.1 404 Not Found
20:13:18.459023 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.56884 > 127.0.0.1.80: Flags [.], ack 1159, win 510, options [nop,nop,TS val 264548163 ecr 264548163], length 0
20:13:33.546721 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.56884 > 127.0.0.1.80: Flags [.], ack 1159, win 512, options [nop,nop,TS val 264563251 ecr 264548163], length 0
20:13:33.546759 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.56884: Flags [.], ack 2711, win 512, options [nop,nop,TS val 264563251 ecr 264548163], length 0
```

