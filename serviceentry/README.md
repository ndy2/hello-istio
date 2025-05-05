## ServiceEntry

outBoundTrafficPolicy 를 REGISTER_ONLY 로 설정 한다음 ServiceEntry 를 등록 하여 Cluster 외부로의 요청을 envoy 가 관리 할 수 있도록 하자!

service entry 는 egress gateway 와 연동하지 않아도 활용 할 수 있지?

```
맞음. ServiceEntry는 Egress Gateway와 연동하지 않아도 단독으로 사용할 수 있음.
정리하자면:
  ServiceEntry는 Istio mesh 외부의 서비스를 mesh 내부에서 인식 가능하도록 등록해주는 리소스임.
  단순히 외부 서비스(예: api.example.com)에 대한 DNS 등록 및 트래픽 제어를 위해서만 사용한다면, Egress Gateway 없이도 충분히 사용 가능함.
  Egress Gateway는 외부 트래픽을 통제된 하나의 출구로 몰아서 보안이나 정책 적용을 강화할 수 있게 해주는 선택적 구성 요소임.
```

## traffic policy 적용 전/ se 적용 전

```
deukyun@deukyun-adsl:~$ k exec -it deploy/curl -- /bin/sh
~ $ curl http://httpbin.org/get
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/8.13.0", 
    "X-Amzn-Trace-Id": "Root=1-681806d7-2911743445bb8d512840d3fe", 
    "X-Envoy-Attempt-Count": "1", 
    "X-Envoy-Peer-Metadata": "ChoKCkNMVVNURVJfSUQSDBoKS3ViZXJuZXRlcwp3CgZMQUJFTFMSbSprCg0KA2FwcBIGGgRjdXJsCikKH3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLW5hbWUSBhoEY3VybAovCiNzZXJ2aWNlLmlzdGlvLmlvL2Nhbm9uaWNhbC1yZXZpc2lvbhIIGgZsYXRlc3QKHwoETkFNRRIXGhVjdXJsLTc1OWRiNTY2OWQteGdmemQKFgoJTkFNRVNQQUNFEgkaB2RlZmF1bHQKSAoFT1dORVISPxo9a3ViZXJuZXRlczovL2FwaXMvYXBwcy92MS9uYW1lc3BhY2VzL2RlZmF1bHQvZGVwbG95bWVudHMvY3VybAoXCg1XT1JLTE9BRF9OQU1FEgYaBGN1cmw=", 
    "X-Envoy-Peer-Metadata-Id": "sidecar~10.1.77.194~curl-759db5669d-xgfzd.default~default.svc.cluster.local"
  }, 
  "origin": "1.221.137.163", 
  "url": "http://httpbin.org/get"
}
~ $ 

deukyun@deukyun-adsl:~$ istioctl pc route deploy/curl
NAME                                                          VHOST NAME                                                    DOMAINS                                              MATCH                  VIRTUAL SERVICE
8000                                                          httpbin.default.svc.cluster.local:8000                        httpbin, httpbin.default + 1 more...                 /*                     
8000                                                          httpbin.foo.svc.cluster.local:8000                            httpbin.foo, 10.152.183.151                          /*                     
9090                                                          kiali.istio-system.svc.cluster.local:9090                     kiali.istio-system, 10.152.183.148                   /*                     
9090                                                          prometheus.istio-system.svc.cluster.local:9090                prometheus.istio-system, 10.152.183.33               /*                     
15014                                                         istiod.istio-system.svc.cluster.local:15014                   istiod.istio-system, 10.152.183.144                  /*                     
20001                                                         kiali.istio-system.svc.cluster.local:20001                    kiali.istio-system, 10.152.183.148                   /*                     
kube-dns.kube-system.svc.cluster.local:9153                   kube-dns.kube-system.svc.cluster.local:9153                   *                                                    /*                     
15010                                                         istiod.istio-system.svc.cluster.local:15010                   istiod.istio-system, 10.152.183.144                  /*                     
istio-ingressgateway.istio-system.svc.cluster.local:15021     istio-ingressgateway.istio-system.svc.cluster.local:15021     *                                                    /*                     
80                                                            2oujlno5e4.execute-api.us-east-1.amazonaws.com:80             2oujlno5e4.execute-api.us-east-1.amazonaws.com       /*                     
80                                                            curl.default.svc.cluster.local:80                             curl, curl.default + 1 more...                       /*                     
80                                                            curl.foo.svc.cluster.local:80                                 curl.foo, 10.152.183.36                              /*                     
80                                                            istio-ingressgateway.istio-system.svc.cluster.local:80        istio-ingressgateway.istio-system, 10.152.183.27     /*                     
                                                              backend                                                       *                                                    /stats/prometheus*     
                                                              backend                                                       *                                                    /healthz/ready*        
inbound|80||                                                  inbound|http|80                                               *                                                    /*                     
inbound|80||                                                  inbound|http|80                                               *                                                    /*                     
InboundPassthroughCluster                                     inbound|http|0                                                *                                                    /*                     
InboundPassthroughCluster                                     inbound|http|0                                                *                                                    /*                     
```

## trafficpolicy 적용 

envoy 에 의해 요청 차단 됨

```
~ $ curl http://httpbin.org/get -v
* Host httpbin.org:80 was resolved.
* IPv6: (none)
* IPv4: 54.243.85.147, 13.216.193.196, 18.205.89.57, 54.236.92.255
*   Trying 54.243.85.147:80...
* Connected to httpbin.org (54.243.85.147) port 80
* using HTTP/1.x
> GET /get HTTP/1.1
> Host: httpbin.org
> User-Agent: curl/8.13.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 502 Bad Gateway
< date: Mon, 05 May 2025 00:32:47 GMT
< server: envoy
< content-length: 0
< 
* Connection #0 to host httpbin.org left intact

deukyun@deukyun-adsl:~$ istioctl pc route deploy/curl
NAME                                                          VHOST NAME                                                    DOMAINS                                              MATCH                  VIRTUAL SERVICE
8000                                                          httpbin.default.svc.cluster.local:8000                        httpbin, httpbin.default + 1 more...                 /*                     
8000                                                          httpbin.foo.svc.cluster.local:8000                            httpbin.foo, 10.152.183.151                          /*                     
8000                                                          block_all                                                     *                                                    /*                     
9090                                                          kiali.istio-system.svc.cluster.local:9090                     kiali.istio-system, 10.152.183.148                   /*                     
9090                                                          prometheus.istio-system.svc.cluster.local:9090                prometheus.istio-system, 10.152.183.33               /*                     
9090                                                          block_all                                                     *                                                    /*                     
15014                                                         istiod.istio-system.svc.cluster.local:15014                   istiod.istio-system, 10.152.183.144                  /*                     
15014                                                         block_all                                                     *                                                    /*                     
20001                                                         kiali.istio-system.svc.cluster.local:20001                    kiali.istio-system, 10.152.183.148                   /*                     
20001                                                         block_all                                                     *                                                    /*                     
kube-dns.kube-system.svc.cluster.local:9153                   kube-dns.kube-system.svc.cluster.local:9153                   *                                                    /*                     
15010                                                         istiod.istio-system.svc.cluster.local:15010                   istiod.istio-system, 10.152.183.144                  /*                     
15010                                                         block_all                                                     *                                                    /*                     
istio-ingressgateway.istio-system.svc.cluster.local:15021     istio-ingressgateway.istio-system.svc.cluster.local:15021     *                                                    /*                     
80                                                            2oujlno5e4.execute-api.us-east-1.amazonaws.com:80             2oujlno5e4.execute-api.us-east-1.amazonaws.com       /*                     
80                                                            curl.default.svc.cluster.local:80                             curl, curl.default + 1 more...                       /*                     
80                                                            curl.foo.svc.cluster.local:80                                 curl.foo, 10.152.183.36                              /*                     
80                                                            istio-ingressgateway.istio-system.svc.cluster.local:80        istio-ingressgateway.istio-system, 10.152.183.27     /*                     
80                                                            block_all                                                     *                                                    /*                     
                                                              backend                                                       *                                                    /stats/prometheus*     
                                                              backend                                                       *                                                    /healthz/ready*        
inbound|80||                                                  inbound|http|80                                               *                                                    /*                     
inbound|80||                                                  inbound|http|80                                               *                                                    /*                     
InboundPassthroughCluster                                     inbound|http|0                                                *                                                    /*                     
InboundPassthroughCluster                                     inbound|http|0                                                *                                                    /*                     
```

route 의 모든 port 에 대하여 `block_all` 이라는 VHOST 가 등록 되었다!

### se 등록 후

```
~ $ curl http://httpbin.org/get
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/8.13.0", 
    "X-Amzn-Trace-Id": "Root=1-681807ed-1609c66d5a007224057a9f07", 
    "X-Envoy-Attempt-Count": "1", 
    "X-Envoy-Decorator-Operation": "httpbin.org:80/*", 
    "X-Envoy-Peer-Metadata": "ChoKCkNMVVNURVJfSUQSDBoKS3ViZXJuZXRlcwp3CgZMQUJFTFMSbSprCg0KA2FwcBIGGgRjdXJsCikKH3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLW5hbWUSBhoEY3VybAovCiNzZXJ2aWNlLmlzdGlvLmlvL2Nhbm9uaWNhbC1yZXZpc2lvbhIIGgZsYXRlc3QKHwoETkFNRRIXGhVjdXJsLTc1OWRiNTY2OWQteGdmemQKFgoJTkFNRVNQQUNFEgkaB2RlZmF1bHQKSAoFT1dORVISPxo9a3ViZXJuZXRlczovL2FwaXMvYXBwcy92MS9uYW1lc3BhY2VzL2RlZmF1bHQvZGVwbG95bWVudHMvY3VybAoXCg1XT1JLTE9BRF9OQU1FEgYaBGN1cmw=", 
    "X-Envoy-Peer-Metadata-Id": "sidecar~10.1.77.194~curl-759db5669d-xgfzd.default~default.svc.cluster.local"
  }, 
  "origin": "1.221.137.163", 
  "url": "http://httpbin.org/get"
}
```

다시 성공!

```
deukyun@deukyun-adsl:~$ istioctl pc route deploy/curl
NAME                                                          VHOST NAME                                                    DOMAINS                                              MATCH                  VIRTUAL SERVICE
8000                                                          httpbin.default.svc.cluster.local:8000                        httpbin, httpbin.default + 1 more...                 /*                     
8000                                                          httpbin.foo.svc.cluster.local:8000                            httpbin.foo, 10.152.183.151                          /*                     
8000                                                          block_all                                                     *                                                    /*                     
9090                                                          kiali.istio-system.svc.cluster.local:9090                     kiali.istio-system, 10.152.183.148                   /*                     
9090                                                          prometheus.istio-system.svc.cluster.local:9090                prometheus.istio-system, 10.152.183.33               /*                     
9090                                                          block_all                                                     *                                                    /*                     
15014                                                         istiod.istio-system.svc.cluster.local:15014                   istiod.istio-system, 10.152.183.144                  /*                     
15014                                                         block_all                                                     *                                                    /*                     
20001                                                         kiali.istio-system.svc.cluster.local:20001                    kiali.istio-system, 10.152.183.148                   /*                     
20001                                                         block_all                                                     *                                                    /*                     
kube-dns.kube-system.svc.cluster.local:9153                   kube-dns.kube-system.svc.cluster.local:9153                   *                                                    /*                     
15010                                                         istiod.istio-system.svc.cluster.local:15010                   istiod.istio-system, 10.152.183.144                  /*                     
15010                                                         block_all                                                     *                                                    /*                     
istio-ingressgateway.istio-system.svc.cluster.local:15021     istio-ingressgateway.istio-system.svc.cluster.local:15021     *                                                    /*                     
80                                                            2oujlno5e4.execute-api.us-east-1.amazonaws.com:80             2oujlno5e4.execute-api.us-east-1.amazonaws.com       /*                     
80                                                            curl.default.svc.cluster.local:80                             curl, curl.default + 1 more...                       /*                     
80                                                            curl.foo.svc.cluster.local:80                                 curl.foo, 10.152.183.36                              /*                     
80                                                            httpbin.org:80                                                httpbin.org                                          /*                     
80                                                            istio-ingressgateway.istio-system.svc.cluster.local:80        istio-ingressgateway.istio-system, 10.152.183.27     /*                     
80                                                            block_all                                                     *                                                    /*                     
                                                              backend                                                       *                                                    /stats/prometheus*     
                                                              backend                                                       *                                                    /healthz/ready*        
inbound|80||                                                  inbound|http|80                                               *                                                    /*                     
inbound|80||                                                  inbound|http|80                                               *                                                    /*                     
InboundPassthroughCluster                                     inbound|http|0                                                *                                                    /*                     
InboundPassthroughCluster                                     inbound|http|0                                                *                                                    /*    
```


`httpbin.org:80` VHOST 가 등록 되었다!
