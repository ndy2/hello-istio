## istio access logging

ref

- https://istio.io/latest/docs/tasks/observability/logs/access-log/

### Mesh Level

```bash
export SOURCE_POD=$(kubectl get pod -l app=curl -o jsonpath={.items..metadata.name})
```

```bash
kubectl exec "$SOURCE_POD" -c curl -- curl -sS -v httpbin:8000/status/418
```

#### 1. default

```bash
istioctl install --set profile=demo \
                 --set meshConfig.accessLogFile=/dev/stdout
```

**curl**

```
[2025-04-18T06:36:33.043Z] "GET /status/418 HTTP/1.1" 418 - via_upstream - "-" 0 13 0 0 "-" "curl/8.13.0" "b0bb62a4-a2ad-92a3-9ba8-5cbc42700ded" "httpbin:8000" "10.1.77.206:8080" outbound|8000||httpbin.default.svc.cluster.local 10.1.77.234:34528 10.152.183.60:8000 10.1.77.234:34968 - default
```

**httpbin**

```
[2025-04-18T06:36:33.044Z] "GET /status/418 HTTP/1.1" 418 - via_upstream - "-" 0 13 0 0 "-" "curl/8.13.0" "b0bb62a4-a2ad-92a3-9ba8-5cbc42700ded" "httpbin:8000" "10.1.77.206:8080" inbound|8080|| 127.0.0.6:42729 10.1.77.206:8080 10.1.77.234:34528 invalid:outbound_.8000_._.httpbin.default.svc.cluster.local default
```

#### 2. json

```bash
istioctl install --set profile=demo \
                 --set meshConfig.accessLogFile=/dev/stdout \
                 --set meshConfig.accessLogEncoding=JSON
```

```json
{
  "upstream_host": "10.1.77.206:8080",
  "upstream_transport_failure_reason": null,
  "upstream_service_time": "3",
  "response_code_details": "via_upstream",
  "authority": "httpbin:8000",
  "bytes_sent": 13,
  "requested_server_name": null,
  "connection_termination_details": null,
  "start_time": "2025-04-18T06:39:19.903Z",
  "x_forwarded_for": null,
  "upstream_local_address": "10.1.77.234:36626",
  "protocol": "HTTP/1.1",
  "downstream_local_address": "10.152.183.60:8000",
  "bytes_received": 0,
  "upstream_cluster": "outbound|8000||httpbin.default.svc.cluster.local;",
  "response_flags": "-",
  "method": "GET",
  "downstream_remote_address": "10.1.77.234:38128",
  "response_code": 418,
  "request_id": "fd52545e-b5b2-99e1-a4ef-bf82390c87e0",
  "user_agent": "curl/8.13.0",
  "duration": 3,
  "path": "/status/418",
  "route_name": "default"
}
```

```json
{
  "upstream_transport_failure_reason": null,
  "upstream_local_address": "127.0.0.6:39177",
  "route_name": "default",
  "bytes_received": 0,
  "path": "/status/418",
  "requested_server_name": "invalid:outbound_.8000_._.httpbin.default.svc.cluster.local",
  "protocol": "HTTP/1.1",
  "bytes_sent": 13,
  "downstream_remote_address": "10.1.77.234:36626",
  "start_time": "2025-04-18T06:39:19.906Z",
  "request_id": "fd52545e-b5b2-99e1-a4ef-bf82390c87e0",
  "connection_termination_details": null,
  "downstream_local_address": "10.1.77.206:8080",
  "upstream_service_time": "0",
  "response_code": 418,
  "response_flags": "-",
  "upstream_cluster": "inbound|8080||;",
  "user_agent": "curl/8.13.0",
  "response_code_details": "via_upstream",
  "method": "GET",
  "upstream_host": "10.1.77.206:8080",
  "duration": 0,
  "x_forwarded_for": null,
  "authority": "httpbin:8000"
}
```

#### 3. json with custom format

- start time
- custom header

```bash
kubectl exec "$SOURCE_POD" -c curl -- curl -sS -v httpbin:8000/status/418 -H "CUSTOM_HEADER: value"
```

```bash
istioctl install --set profile=demo \
                 --set meshConfig.accessLogFile=/dev/stdout \
                 --set meshConfig.accessLogEncoding=JSON \
                 --set meshConfig.accessLogFormat="{\"start_time\": \"%START_TIME%\", \"custom_header\": \"%REQ(CUSTOM_HEADER)%\"}"
```

```json
{
  "custom_header": "value",
  "start_time": "2025-04-18T06:49:06.312Z"
}
```


### Namespace Level

```bash
kubens foo
kubectl create -f ef-merge.yaml
kubectl create -f curl.yaml
kubectl create -f httpbin.yaml
```

httpbin

```log
[2025-04-19T05:56:05.414Z] "GET /status/418 HTTP/1.1" 418 - via_upstream - "-" 0 13 0 0 "-" "curl/8.13.0" "6310e935-7d2f-9da3-bcf8-e08f0b46ad1f" "httpbin:8000" "10.244.0.13:8080" inbound|8080|| 127.0.0.6:56265 10.244.0.13:8080 10.244.0.12:36446 outbound_.8000_._.httpbin.foo.svc.cluster.local default
{"start_time":"2025-04-19T05:56:05.414Z","custom_header":"value"}
```

### Telemetry API

ref.
- https://istio.io/latest/docs/reference/config/telemetry/
- https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-ExtensionProvider-EnvoyFileAccessLogProvider

```bash
istioctl install -f istiooperator.yaml
kubectl create -f telemetry.yaml
```

=> foo ns 에만 적절한 format 줄 수 있다!

```
{"message":"","status":418}
{"message":"","status":418}
```