# sidecar

```bash
export SOURCE_POD=$(kubectl get pod -l app=curl -o jsonpath={.items..metadata.name})
```

```bash
kubectl exec "$SOURCE_POD" -c curl -- curl -sS -v httpbin:8000/status/418
```

---

```
╭─deukyun@namdeug-yun-ui-Macmini ~
╰─$ istioctl ps deploy/curl
Clusters Match
Listeners Match
Routes Match (RDS last loaded at Sat, 03 May 2025 19:15:26 KST)
╭─deukyun@namdeug-yun-ui-Macmini ~
╰─$ istioctl ps                                                                                                                                                   127 ↵
NAME                                                   CLUSTER        CDS                LDS                EDS               RDS                ECDS        ISTIOD                      VERSION
curl-594d84cbb6-nqtlp.foo                              Kubernetes     SYNCED (111s)      SYNCED (111s)      SYNCED (111s)     SYNCED (111s)      IGNORED     istiod-7c569b5995-w5d8k     1.23.0
curl-74c989df8d-2xlx6.default                          Kubernetes     SYNCED (3m10s)     SYNCED (3m10s)     SYNCED (3m8s)     SYNCED (3m10s)     IGNORED     istiod-7c569b5995-w5d8k     1.23.0
httpbin-5fd45fcd6f-hjzt5.foo                           Kubernetes     SYNCED (111s)      SYNCED (111s)      SYNCED (111s)     SYNCED (111s)      IGNORED     istiod-7c569b5995-w5d8k     1.23.0
httpbin-686d6fc899-kq8db.default                       Kubernetes     SYNCED (3m8s)      SYNCED (3m8s)      SYNCED (3m8s)     SYNCED (3m8s)      IGNORED     istiod-7c569b5995-w5d8k     1.23.0
istio-egressgateway-789bc6c945-zlw96.istio-system      Kubernetes     SYNCED (111s)      SYNCED (111s)      SYNCED (111s)     SYNCED (111s)      IGNORED     istiod-7c569b5995-w5d8k     1.23.0
istio-ingressgateway-545f887965-4fckw.istio-system     Kubernetes     SYNCED (111s)      SYNCED (111s)      SYNCED (111s)     IGNORED            IGNORED     istiod-7c569b5995-w5d8k     1.23.0
```

---

before sidecar

```
╭─deukyun@namdeug-yun-ui-Macmini ~
╰─$ istioctl pc cluster deploy/curl
SERVICE FQDN                                            PORT      SUBSET     DIRECTION     TYPE             DESTINATION RULE
                                                        80        -          inbound       ORIGINAL_DST
BlackHoleCluster                                        -         -          -             STATIC
InboundPassthroughCluster                               -         -          -             ORIGINAL_DST
PassthroughCluster                                      -         -          -             ORIGINAL_DST
agent                                                   -         -          -             STATIC
curl.default.svc.cluster.local                          80        -          outbound      EDS
curl.foo.svc.cluster.local                              80        -          outbound      EDS
httpbin.default.svc.cluster.local                       8000      -          outbound      EDS
httpbin.foo.svc.cluster.local                           8000      -          outbound      EDS
httpbin.org                                             80        -          outbound      STRICT_DNS       egress-tls.default
httpbin.org                                             443       -          outbound      STRICT_DNS       egress-tls.default
istio-egressgateway.istio-system.svc.cluster.local      80        -          outbound      EDS
istio-egressgateway.istio-system.svc.cluster.local      443       -          outbound      EDS
istio-ingressgateway.istio-system.svc.cluster.local     80        -          outbound      EDS
istio-ingressgateway.istio-system.svc.cluster.local     443       -          outbound      EDS
istio-ingressgateway.istio-system.svc.cluster.local     15021     -          outbound      EDS
istio-ingressgateway.istio-system.svc.cluster.local     15443     -          outbound      EDS
istio-ingressgateway.istio-system.svc.cluster.local     31400     -          outbound      EDS
istiod.istio-system.svc.cluster.local                   443       -          outbound      EDS
istiod.istio-system.svc.cluster.local                   15010     -          outbound      EDS
istiod.istio-system.svc.cluster.local                   15012     -          outbound      EDS
istiod.istio-system.svc.cluster.local                   15014     -          outbound      EDS
kube-dns.kube-system.svc.cluster.local                  53        -          outbound      EDS
kube-dns.kube-system.svc.cluster.local                  9153      -          outbound      EDS
kubernetes.default.svc.cluster.local                    443       -          outbound      EDS
prometheus_stats                                        -         -          -             STATIC
sds-grpc                                                -         -          -             STATIC
xds-grpc                                                -         -          -             STATIC
╭─deukyun@namdeug-yun-ui-Macmini ~
╰─$ istioctl pc route deploy/curl
NAME                                                          VHOST NAME                                                    DOMAINS                                             MATCH                  VIRTUAL SERVICE
80                                                            curl.default.svc.cluster.local:80                             curl.default, 10.96.137.103                         /*
80                                                            curl.foo.svc.cluster.local:80                                 curl, curl.foo + 1 more...                          /*
80                                                            httpbin.org:80                                                httpbin.org                                         /*
80                                                            istio-egressgateway.istio-system.svc.cluster.local:80         istio-egressgateway.istio-system, 10.96.103.231     /*
80                                                            istio-ingressgateway.istio-system.svc.cluster.local:80        istio-ingressgateway.istio-system, 10.96.87.4       /*
istio-ingressgateway.istio-system.svc.cluster.local:15021     istio-ingressgateway.istio-system.svc.cluster.local:15021     *                                                   /*
kube-dns.kube-system.svc.cluster.local:9153                   kube-dns.kube-system.svc.cluster.local:9153                   *                                                   /*
8000                                                          httpbin.default.svc.cluster.local:8000                        httpbin.default, 10.96.46.133                       /*
8000                                                          httpbin.foo.svc.cluster.local:8000                            httpbin, httpbin.foo + 1 more...                    /*
15010                                                         istiod.istio-system.svc.cluster.local:15010                   istiod.istio-system, 10.96.4.130                    /*
15014                                                         istiod.istio-system.svc.cluster.local:15014                   istiod.istio-system, 10.96.4.130                    /*
                                                              backend                                                       *                                                   /stats/prometheus*
                                                              backend                                                       *                                                   /healthz/ready*
inbound|80||                                                  inbound|http|80                                               *                                                   /*
InboundPassthroughCluster                                     inbound|http|0                                                *                                                   /*
inbound|80||                                                  inbound|http|80                                               *                                                   /*
InboundPassthroughCluster                                     inbound|http|0                                                *                                                   /*
```

```
╭─deukyun@namdeug-yun-ui-Macmini ~/Desktop/hello-istio/sidecar/manifests ‹main●› 
╰─$ kubectl exec "$SOURCE_POD" -c curl -- curl -sS -v httpbin.foo:8000/status/418
...

요청 성공
```


after sidecar - foo 및 istio-system 관련 ns 로의 envoy route/cluster 설정이 사라졌다!

```
╭─deukyun@namdeug-yun-ui-Macmini ~
╰─$ istioctl pc cluster deploy/curl
SERVICE FQDN                             PORT     SUBSET     DIRECTION     TYPE             DESTINATION RULE
                                         80       -          inbound       ORIGINAL_DST
BlackHoleCluster                         -        -          -             STATIC
InboundPassthroughCluster                -        -          -             ORIGINAL_DST
PassthroughCluster                       -        -          -             ORIGINAL_DST
agent                                    -        -          -             STATIC
curl.default.svc.cluster.local           80       -          outbound      EDS
httpbin.default.svc.cluster.local        8000     -          outbound      EDS
httpbin.org                              80       -          outbound      STRICT_DNS       egress-tls.default
httpbin.org                              443      -          outbound      STRICT_DNS       egress-tls.default
kubernetes.default.svc.cluster.local     443      -          outbound      EDS
prometheus_stats                         -        -          -             STATIC
sds-grpc                                 -        -          -             STATIC
xds-grpc                                 -        -          -             STATIC

╭─deukyun@namdeug-yun-ui-Macmini ~
╰─$ istioctl pc route deploy/curl
NAME                          VHOST NAME                                 DOMAINS                                  MATCH                  VIRTUAL SERVICE
80                            curl.default.svc.cluster.local:80          curl, curl.default + 1 more...           /*
80                            httpbin.org:80                             httpbin.org                              /*
8000                          httpbin.default.svc.cluster.local:8000     httpbin, httpbin.default + 1 more...     /*
                              backend                                    *                                        /stats/prometheus*
InboundPassthroughCluster     inbound|http|0                             *                                        /*
                              backend                                    *                                        /healthz/ready*
InboundPassthroughCluster     inbound|http|0                             *                                        /*
inbound|80||                  inbound|http|80                            *                                        /*
inbound|80||                  inbound|http|80                            *                                        /*
```

register-only 로 동작하면 등록되지 않은 outbound traffic 이 차단된다!

```
╭─deukyun@namdeug-yun-ui-Macmini ~/Desktop/hello-istio/sidecar/manifests ‹main●› 
╰─$ kubectl exec "$SOURCE_POD" -c curl -- curl -sS -v httpbin.foo:8000/status/418



...
* Request completely sent off
< HTTP/1.1 502 Bad Gateway
< date: Sat, 03 May 2025 10:34:54 GMT
< server: envoy
< content-length: 0
```
