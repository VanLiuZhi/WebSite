---
weight: 1
title: "K8s Kong"
subtitle: "K8s Kong"
date: 2020-12-17T09:48:48+08:00
lastmod: 2020-12-17T09:48:48+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Kong"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: 
categories: 

lightgallery: true

toc:
  # 自动展开目录
  auto: false
---

kong 多网关部署实践

<!--more-->

## 概要

多网关隔离了数据。虽然单网关不同host也能隔离资源，但是多网关的模式从入口上就隔离了资源

采用官方最新的版本部署

kong-ingress-controller:1.1
kong:2.2

通过命名空间划分资源，PgSql也需要部署两份，通过CONTROLLER_INGRESS_CLASS绑定ingress

## kong和ingress的一些总结

kong 1.1 版本，ingress如果不设置 class, 不会被管理后台统计到，但是旧的路由依然能用（能访问，不被kong监听到），然后新加入不指定class也是不行(不能访问，不被kong监听到)

k8s 1.14 ingress 的 address 就算集群中没有ingress-controller，address也不会被清空

部署多个Ingress控制器而不指定注释会导致两个控制器都争相满足Ingress

不指定注释将导致多个入口控制器声明相同的入口（多个kong-controller相同）。设置与任何现有入口控制器的类都不匹配的值将导致所有入口控制器忽略该入口。（kong-controller 的class 和 ingress不匹配，ingress被忽略）

## INGRESS_CLASS

ingress-controller可以通过配置INGRESS_CLASS，当ingress的class和ingress-controller的class匹配上时，该ingress才会被纳入配置

对于kong网关，默认calss是kong，修改配置注入环境变量

```yaml
- name: CONTROLLER_INGRESS_CLASS
  value: "kong-lee"
```

在ingress上声明这个配置，指明由哪个ingress-controller处理

```yaml
annotations:
  kubernetes.io/ingress.class: "kong-lee"
```

## pg docker 部署

docker run  -di --name=eos_postgre1 -p 5433:5432 --restart=always -e TZ=Asia/Shanghai -e POSTGRES_USER=kong -e POSTGRES_PASSWORD=kong -e POSTGRES_DB=kong -v /data/docker/postgres5433/data:/var/lib/postgresql/data hub.eos.h3c.com/base/postgres:9.5 

## 绑定80端口

官方1.1版本默认是绑定8000端口的，而且是pod内部的端口

从service不难看出，默认是 `type: LoadBalancer` ，也就是采用外部负载均衡，而且annotations也声明了AWS的配置。这种一般是针对云厂商的，由于我们是私有云，也没有F5等外部负载均衡器。所以需要修改这个配置，让端口绑定到80上

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
```

修改这句

```yaml
- name: KONG_PROXY_LISTEN
  value: 0.0.0.0:80, 0.0.0.0:8443 ssl http2
```

但是启动服务发现 `bind() to 0.0.0.0:80 failed (13: Permission denied)` 的错误，也就是没有权限，一般普通用户是没有绑定1024以下端口的权限的

kong的镜像是用kong用户启动的，这里有多种解决方案

1. 使用root用户来运行
2. Linux capabilities 来设置权限

如果怕root用户权限太高，可以考虑2的方案，这里我们采用1，securityContext设置pod安全相关配置

```yaml
securityContext:
  runAsUser: 0
```

由于我们是要绑定到宿主机，所以采用hostNetwork模式，并且指定部署节点做为网关地址

```yaml
hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
  nodeSelector:
    kubernetes.io/hostname: k8s-05
```

hostNetwork的延伸：

在使用hostNetwork模式，并且有端口需要绑定的时候，deployment的滚动更新会失效。猜测因为旧版本占用了宿主机端口，新的pod无法绑定导致一直处于pending状态

## ingress paths 访问404

通常我们的ingress会这样配置，通过域名访问，然后匹配不同的path到对应的服务

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-app
  namespace: eos-demos
spec:
  rules:
  - host: myapp.h3c.com
    http:
      paths:
      - path: /eos/
        backend:
          serviceName: echo
          servicePort: 80
```

但是在kong1.1版本中，发现这样配置访问会导致404。查阅官方文档和论坛后发现，新版本的访问策略改变了

假如当前服务是 echo，有url `healthy`可以访问到

旧版本的部署方式path用 `/eos/`，通过 `myapp.h3c.com/eos/healthy` 可以访问到

但是新版本把path当成服务的路由的一部分
旧版本：`myapp.h3c.com/eos/healthy` --> `myapp.h3c.com/healthy` (实际请求路径)
新版本: `myapp.h3c.com/eos/healthy` --> `myapp.h3c.com/eos/healthy` (实际请求路径)

这种效果不是我们想要的，我们主要就是通过同一个域名和path划分服务

有两种方案可以解决这个问题：

1. konghq.com/strip-path: "true"

在annotations中设置，可以改变策略

2. 通过KongIngress配置

通过创建kong 的 CRD 资源对象 KongIngress，可以配置很多策略，比如`strip_path: true`，还有其它很多特性

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongIngress
metadata:
  name: kong-override
  namespace: kong2
route:
  strip_path: true
```

在ingress中，通过`konghq.com/override: "kong-override"` 关联即可

需要注意的是，KongIngress必须要和ingress在同一个命名空间下，否则关联会失败


总结：

总体来说方案1会比较简单，在0.8版本后支持这个配置。如果没有特别的定制要求不需要去使用KongIngress

## lua源码修改

我们有些需求无法满足的时候，可以通过修改lua源码来实现。重新打成镜像即可

如果从源码构建，官方的方式比较复杂。官方的Dockerfile是从rpm包构建的。也就是我们需要先传rpm

涉及到的项目
```
Kong/docker-kong        镜像构建
Kong/kong-build-tools   源码编译工具
```

如果只是改lua脚本不需要这么复杂的方式，可以从官方镜像继承，然后把修改后的lua脚本替换

```
lua不需要编译，直接由解释器执行即可。python会先翻译成中间文件，也就是字节码，之后再由解释器执行
```

dockerfile构建，handler.lua就是我们修改过的源码

```dockerfile
FROM hub.eos.h3c.com/kong/kong:2.2

LABEL maintainer="lys3415"

COPY ./handler.lua /usr/local/share/lua/5.1/kong/plugins/cors/
```

修改部分，kong2.2版本

```lua
function CreateUUID()
  local template ="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  d = io.open("/dev/urandom", "r"):read(4)
  math.randomseed(os.time() + d:byte(1) + (d:byte(2) * 256) + (d:byte(3) * 65536) + (d:byte(4) * 4294967296))
  return string.gsub(template, "x", function (c)
        local v = (c == "x") and math.random(0, 0xf) or math.random(8, 0xb)
        return string.format("%x", v)
        end)
end

function CorsHandler:access(conf)
  if kong.request.get_method() ~= "OPTIONS"
     or not kong.request.get_header("Origin")
     or not kong.request.get_header("Access-Control-Request-Method")
  then
    kong.service.request.set_header("trace",CreateUUID())
    local konghost = os.getenv("HOSTNAME")
    if konghost then
      kong.response.set_header("Test", "true")
    else
      kong.response.set_header("Test", "false")
    end
    kong.response.set_header("Kong-Host", konghost)
    return
  end
... 省略
```

## docker 启动 pgsql

docker run  -di --name=eos_postgre1 -p 5433:5432 --restart=always -e TZ=Asia/Shanghai -e POSTGRES_USER=kong -e POSTGRES_PASSWORD=kong -e POSTGRES_DB=kong -v /data/docker/postgres5433/data:/var/lib/postgresql/data hub.eos.h3c.com/base/postgres:9.5 

## 参考

官方文档

https://docs.konghq.com/kubernetes-ingress-controller/1.1.x/guides/using-kongingress-resource/

https://cloud.tencent.com/developer/article/1534676

## yaml

创建kong和kong2的命名空间，创建crd。调整镜像和节点等配置，部署kong和kong2

konga也需要独立部署，展示了kong2的konga的部署

CRD

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: kongclusterplugins.configuration.konghq.com
spec:
  additionalPrinterColumns:
    - JSONPath: .plugin
      description: Name of the plugin
      name: Plugin-Type
      type: string
    - JSONPath: .metadata.creationTimestamp
      description: Age
      name: Age
      type: date
    - JSONPath: .disabled
      description: Indicates if the plugin is disabled
      name: Disabled
      priority: 1
      type: boolean
    - JSONPath: .config
      description: Configuration of the plugin
      name: Config
      priority: 1
      type: string
  group: configuration.konghq.com
  names:
    kind: KongClusterPlugin
    plural: kongclusterplugins
    shortNames:
      - kcp
  scope: Cluster
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      properties:
        config:
          type: object
        configFrom:
          properties:
            secretKeyRef:
              properties:
                key:
                  type: string
                name:
                  type: string
                namespace:
                  type: string
              required:
                - name
                - namespace
                - key
              type: object
          type: object
        disabled:
          type: boolean
        plugin:
          type: string
        protocols:
          items:
            enum:
              - http
              - https
              - grpc
              - grpcs
              - tcp
              - tls
            type: string
          type: array
        run_on:
          enum:
            - first
            - second
            - all
          type: string
      required:
        - plugin
  version: v1
---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: kongconsumers.configuration.konghq.com
spec:
  additionalPrinterColumns:
    - JSONPath: .username
      description: Username of a Kong Consumer
      name: Username
      type: string
    - JSONPath: .metadata.creationTimestamp
      description: Age
      name: Age
      type: date
  group: configuration.konghq.com
  names:
    kind: KongConsumer
    plural: kongconsumers
    shortNames:
      - kc
  scope: Namespaced
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      properties:
        credentials:
          items:
            type: string
          type: array
        custom_id:
          type: string
        username:
          type: string
  version: v1
---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: kongingresses.configuration.konghq.com
spec:
  group: configuration.konghq.com
  names:
    kind: KongIngress
    plural: kongingresses
    shortNames:
      - ki
  scope: Namespaced
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      properties:
        proxy:
          properties:
            connect_timeout:
              minimum: 0
              type: integer
            path:
              pattern: ^/.*$
              type: string
            protocol:
              enum:
                - http
                - https
                - grpc
                - grpcs
                - tcp
                - tls
              type: string
            read_timeout:
              minimum: 0
              type: integer
            retries:
              minimum: 0
              type: integer
            write_timeout:
              minimum: 0
              type: integer
          type: object
        route:
          properties:
            headers:
              additionalProperties:
                items:
                  type: string
                type: array
              type: object
            https_redirect_status_code:
              type: integer
            methods:
              items:
                type: string
              type: array
            path_handling:
              enum:
                - v0
                - v1
              type: string
            preserve_host:
              type: boolean
            protocols:
              items:
                enum:
                  - http
                  - https
                  - grpc
                  - grpcs
                  - tcp
                  - tls
                type: string
              type: array
            regex_priority:
              type: integer
            snis:
              items:
                type: string
              type: array
            strip_path:
              type: boolean
        upstream:
          properties:
            algorithm:
              enum:
                - round-robin
                - consistent-hashing
                - least-connections
              type: string
            hash_fallback:
              type: string
            hash_fallback_header:
              type: string
            hash_on:
              type: string
            hash_on_cookie:
              type: string
            hash_on_cookie_path:
              type: string
            hash_on_header:
              type: string
            healthchecks:
              properties:
                active:
                  properties:
                    concurrency:
                      minimum: 1
                      type: integer
                    healthy:
                      properties:
                        http_statuses:
                          items:
                            type: integer
                          type: array
                        interval:
                          minimum: 0
                          type: integer
                        successes:
                          minimum: 0
                          type: integer
                      type: object
                    http_path:
                      pattern: ^/.*$
                      type: string
                    timeout:
                      minimum: 0
                      type: integer
                    unhealthy:
                      properties:
                        http_failures:
                          minimum: 0
                          type: integer
                        http_statuses:
                          items:
                            type: integer
                          type: array
                        interval:
                          minimum: 0
                          type: integer
                        tcp_failures:
                          minimum: 0
                          type: integer
                        timeout:
                          minimum: 0
                          type: integer
                      type: object
                  type: object
                passive:
                  properties:
                    healthy:
                      properties:
                        http_statuses:
                          items:
                            type: integer
                          type: array
                        interval:
                          minimum: 0
                          type: integer
                        successes:
                          minimum: 0
                          type: integer
                      type: object
                    unhealthy:
                      properties:
                        http_failures:
                          minimum: 0
                          type: integer
                        http_statuses:
                          items:
                            type: integer
                          type: array
                        interval:
                          minimum: 0
                          type: integer
                        tcp_failures:
                          minimum: 0
                          type: integer
                        timeout:
                          minimum: 0
                          type: integer
                      type: object
                  type: object
                threshold:
                  type: integer
              type: object
            host_header:
              type: string
            slots:
              minimum: 10
              type: integer
          type: object
  version: v1
---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: kongplugins.configuration.konghq.com
spec:
  additionalPrinterColumns:
    - JSONPath: .plugin
      description: Name of the plugin
      name: Plugin-Type
      type: string
    - JSONPath: .metadata.creationTimestamp
      description: Age
      name: Age
      type: date
    - JSONPath: .disabled
      description: Indicates if the plugin is disabled
      name: Disabled
      priority: 1
      type: boolean
    - JSONPath: .config
      description: Configuration of the plugin
      name: Config
      priority: 1
      type: string
  group: configuration.konghq.com
  names:
    kind: KongPlugin
    plural: kongplugins
    shortNames:
      - kp
  scope: Namespaced
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      properties:
        config:
          type: object
        configFrom:
          properties:
            secretKeyRef:
              properties:
                key:
                  type: string
                name:
                  type: string
              required:
                - name
                - key
              type: object
          type: object
        disabled:
          type: boolean
        plugin:
          type: string
        protocols:
          items:
            enum:
              - http
              - https
              - grpc
              - grpcs
              - tcp
              - tls
            type: string
          type: array
        run_on:
          enum:
            - first
            - second
            - all
          type: string
      required:
        - plugin
  version: v1
---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tcpingresses.configuration.konghq.com
spec:
  additionalPrinterColumns:
    - JSONPath: .status.loadBalancer.ingress[*].ip
      description: Address of the load balancer
      name: Address
      type: string
    - JSONPath: .metadata.creationTimestamp
      description: Age
      name: Age
      type: date
  group: configuration.konghq.com
  names:
    kind: TCPIngress
    plural: tcpingresses
  scope: Namespaced
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      properties:
        apiVersion:
          type: string
        kind:
          type: string
        metadata:
          type: object
        spec:
          properties:
            rules:
              items:
                properties:
                  backend:
                    properties:
                      serviceName:
                        type: string
                      servicePort:
                        format: int32
                        type: integer
                    type: object
                  host:
                    type: string
                  port:
                    format: int32
                    type: integer
                type: object
              type: array
            tls:
              items:
                properties:
                  hosts:
                    items:
                      type: string
                    type: array
                  secretName:
                    type: string
                type: object
              type: array
          type: object
        status:
          type: object
  version: v1beta1
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
```

kong 部署

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kong-serviceaccount
  namespace: kong
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: kong-ingress-clusterrole
rules:
  - apiGroups:
      - ""
    resources:
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - networking.k8s.io
      - extensions
      - networking.internal.knative.dev
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - networking.k8s.io
      - extensions
      - networking.internal.knative.dev
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - configuration.konghq.com
    resources:
      - tcpingresses/status
    verbs:
      - update
  - apiGroups:
      - configuration.konghq.com
    resources:
      - kongplugins
      - kongclusterplugins
      - kongcredentials
      - kongconsumers
      - kongingresses
      - tcpingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
      - get
      - update
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kong-ingress-clusterrole-nisa-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kong-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: kong-serviceaccount
    namespace: kong
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
  name: kong-proxy
  namespace: kong
spec:
  ports:
    - name: proxy
      port: 80
      protocol: TCP
      targetPort: 8000
    - name: proxy-ssl
      port: 443
      protocol: TCP
      targetPort: 8443
  selector:
    app: ingress-kong
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: kong-validation-webhook
  namespace: kong
spec:
  ports:
    - name: webhook
      port: 443
      protocol: TCP
      targetPort: 8080
  selector:
    app: ingress-kong
---
apiVersion: v1
kind: Service
metadata:
  name: kong-ingress-controller
  namespace: kong
spec:
  type: NodePort
  ports:
    - name: kong-admin
      port: 8001
      targetPort: 8001
      #nodePort: 30001
      protocol: TCP
  selector:
    app: ingress-kong
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: kong
spec:
  ports:
    - name: pgql
      port: 5432
      protocol: TCP
      targetPort: 5432
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ingress-kong
  name: ingress-kong
  namespace: kong
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-kong
  template:
    metadata:
      annotations:
        kuma.io/gateway: enabled
        prometheus.io/port: "8100"
        prometheus.io/scrape: "true"
        traffic.sidecar.istio.io/includeInboundPorts: ""
      labels:
        app: ingress-kong
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      nodeSelector:
        kubernetes.io/hostname: k8s-07
      containers:
        - env:
            - name: KONG_DATABASE # 要使用的数据库类型，不是名称
              value: postgres
            - name: KONG_PG_HOST
              value: postgres
            - name: KONG_PG_PASSWORD
              value: kong
            - name: KONG_PROXY_LISTEN
              value: 0.0.0.0:80, 0.0.0.0:8443 ssl http2
            - name: KONG_PORT_MAPS
              value: 80:8000, 443:8443
            - name: KONG_ADMIN_LISTEN
              value: 0.0.0.0:8001, 127.0.0.1:8444 ssl
            - name: KONG_STATUS_LISTEN
              value: 0.0.0.0:8100
            - name: KONG_NGINX_WORKER_PROCESSES
              value: "2"
            - name: KONG_ADMIN_ACCESS_LOG
              value: /dev/stdout
            - name: KONG_ADMIN_ERROR_LOG
              value: /dev/stderr
            - name: KONG_PROXY_ERROR_LOG
              value: /dev/stderr
          image: hub.eos.h3c.com/kong/kong:2.2
          # pod 安全设置
          #          securityContext:
          #            capabilities:
          #              add:
          #                - NET_BIND_SERVICE
          securityContext:
            runAsUser: 0
          lifecycle:
            preStop:
              exec:
                command:
                  - /bin/sh
                  - -c
                  - kong quit
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /status
              port: 8100
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          name: proxy
          ports:
            - containerPort: 8000
              name: proxy
              protocol: TCP
            - containerPort: 8443
              name: proxy-ssl
              protocol: TCP
            - containerPort: 8100
              name: metrics
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /status
              port: 8100
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
        - env:
            - name: CONTROLLER_INGRESS_CLASS
              value: "kong-liuzhi"
            - name: CONTROLLER_KONG_ADMIN_URL
              value: https://127.0.0.1:8444
            - name: CONTROLLER_KONG_ADMIN_TLS_SKIP_VERIFY
              value: "true"
            - name: CONTROLLER_PUBLISH_SERVICE
              value: kong/kong-proxy
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
          image: hub.eos.h3c.com/kong/kong-ingress-controller:1.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          name: ingress-controller
          ports:
            - containerPort: 8080
              name: webhook
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
      initContainers:
        - command:
            - /bin/sh
            - -c
            - while true; do kong migrations list; if [[ 0 -eq $? ]]; then exit 0; fi; sleep 2;  done;
          env:
            - name: KONG_PG_HOST
              value: postgres
            - name: KONG_PG_PASSWORD
              value: kong
          image: hub.eos.h3c.com/kong/kong:2.2
          name: wait-for-migrations
      serviceAccountName: kong-serviceaccount
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: kong
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  serviceName: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-04
      containers:
        - env:
            - name: POSTGRES_USER
              value: kong
            - name: POSTGRES_PASSWORD
              value: kong
            - name: POSTGRES_DB
              value: kong
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          image: hub.eos.h3c.com/base/postgres:9.5
          name: postgres
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: datadirpg
              subPath: pgdata
              readOnly: false
      volumes:
        - name: datadirpg
          emptyDir: {}
      terminationGracePeriodSeconds: 60
  # volumeClaimTemplates:                   # 用于持久化的模板
  #   - metadata:
  #       name: datadirpg
  #       annotations:
  #         volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
  #     spec:
  #       accessModes:
  #         - ReadWriteOnce
  #       resources:
  #         requests:
  #           storage: 1Gi
  # volumeClaimTemplates:
  #   - metadata:
  #       name: datadir
  #     spec:
  #       accessModes: ["ReadWriteOnce"]
  #       storageClassName: "eos-ceph"
  #       resources:
  #         requests:
  #           storage: 1Gi
#  volumeClaimTemplates:
#    - metadata:
#        name: datadir
#      spec:
#        accessModes:
#          - ReadWriteOnce
#        resources:
#          requests:
#            storage: 1Gi
---
apiVersion: batch/v1
kind: Job
metadata:
  name: kong-migrations
  namespace: kong
spec:
  template:
    metadata:
      name: kong-migrations
    spec:
      containers:
        - command:
            - /bin/sh
            - -c
            - kong migrations bootstrap
          env:
            - name: KONG_PG_PASSWORD
              value: kong
            - name: KONG_PG_HOST
              value: postgres
            - name: KONG_PG_PORT
              value: "5432"
          image: hub.eos.h3c.com/kong/kong:2.2
          name: kong-migrations
      initContainers:
        - command:
            - /bin/sh
            - -c
            - until nc -zv $KONG_PG_HOST $KONG_PG_PORT -w1; do echo 'waiting for db'; sleep 1; done
          env:
            - name: KONG_PG_HOST
              value: postgres
            - name: KONG_PG_PORT
              value: "5432"
          image: hub.eos.h3c.com/kong/busybox
          name: wait-for-postgres
      restartPolicy: OnFailure
```

kong2 部署

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kong-serviceaccount2
  namespace: kong2
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: kong-ingress-clusterrole2
rules:
  - apiGroups:
      - ""
    resources:
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - networking.k8s.io
      - extensions
      - networking.internal.knative.dev
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - networking.k8s.io
      - extensions
      - networking.internal.knative.dev
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - configuration.konghq.com
    resources:
      - tcpingresses/status
    verbs:
      - update
  - apiGroups:
      - configuration.konghq.com
    resources:
      - kongplugins
      - kongclusterplugins
      - kongcredentials
      - kongconsumers
      - kongingresses
      - tcpingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
      - get
      - update
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kong-ingress-clusterrole-nisa-binding2
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kong-ingress-clusterrole2
subjects:
  - kind: ServiceAccount
    name: kong-serviceaccount2
    namespace: kong2
---
apiVersion: v1
kind: Service
metadata:
  #  annotations:
  #    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
  #    service.beta.kubernetes.io/aws-load-balancer-type: nlb
  name: kong-proxy2
  namespace: kong2
spec:
  ports:
    - name: proxy
      port: 80
      protocol: TCP
      targetPort: 8000
    - name: proxy-ssl
      port: 443
      protocol: TCP
      targetPort: 8443
  selector:
    app: ingress-kong2
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: kong-validation-webhook2
  namespace: kong2
spec:
  ports:
    - name: webhook
      port: 443
      protocol: TCP
      targetPort: 8080
  selector:
    app: ingress-kong2
---
apiVersion: v1
kind: Service
metadata:
  name: kong-ingress-controller2
  namespace: kong2
spec:
  type: NodePort
  ports:
    - name: kong-admin
      port: 8001
      targetPort: 8001
      #nodePort: 30001
      protocol: TCP
  selector:
    app: ingress-kong2
---
apiVersion: v1
kind: Service
metadata:
  name: postgres2
  namespace: kong2
spec:
  ports:
    - name: pgql
      port: 5432
      protocol: TCP
      targetPort: 5432
  selector:
    app: postgres2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ingress-kong2
  name: ingress-kong2
  namespace: kong2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-kong2
  template:
    metadata:
      annotations:
        kuma.io/gateway: enabled
        prometheus.io/port: "8100"
        prometheus.io/scrape: "true"
        traffic.sidecar.istio.io/includeInboundPorts: ""
      labels:
        app: ingress-kong2
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      nodeSelector:
        kubernetes.io/hostname: k8s-05
      containers:
        - env:
            - name: KONG_DATABASE
              value: postgres
            - name: KONG_PG_HOST
              value: postgres2
            - name: KONG_PG_PASSWORD
              value: kong
            - name: KONG_PROXY_LISTEN
              value: 0.0.0.0:80, 0.0.0.0:8443 ssl http2
            - name: KONG_PORT_MAPS
              value: 80:8000, 443:8443
            - name: KONG_ADMIN_LISTEN
              value: 0.0.0.0:8001, 127.0.0.1:8444 ssl
            - name: KONG_STATUS_LISTEN
              value: 0.0.0.0:8100
            - name: KONG_NGINX_WORKER_PROCESSES
              value: "2"
            - name: KONG_ADMIN_ACCESS_LOG
              value: /dev/stdout
            - name: KONG_ADMIN_ERROR_LOG
              value: /dev/stderr
            - name: KONG_PROXY_ERROR_LOG
              value: /dev/stderr
          image: hub.eos.h3c.com/kong/kong:2.2
          # pod 安全设置
          #          securityContext:
          #            capabilities:
          #              add:
          #                - NET_BIND_SERVICE
          securityContext:
            runAsUser: 0
          lifecycle:
            preStop:
              exec:
                command:
                  - /bin/sh
                  - -c
                  - kong quit
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /status
              port: 8100
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          name: proxy
          ports:
            - containerPort: 8000
              name: proxy
              protocol: TCP
            - containerPort: 8443
              name: proxy-ssl
              protocol: TCP
            - containerPort: 8100
              name: metrics
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /status
              port: 8100
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
        - env:
            - name: CONTROLLER_INGRESS_CLASS
              value: "kong-lee"
            - name: CONTROLLER_KONG_ADMIN_URL
              value: https://127.0.0.1:8444
            - name: CONTROLLER_KONG_ADMIN_TLS_SKIP_VERIFY
              value: "true"
            - name: CONTROLLER_PUBLISH_SERVICE
              value: kong2/kong-proxy2
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
          image: hub.eos.h3c.com/kong/kong-ingress-controller:1.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          name: ingress-controller
          ports:
            - containerPort: 8080
              name: webhook
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
      initContainers:
        - command:
            - /bin/sh
            - -c
            - while true; do kong migrations list; if [[ 0 -eq $? ]]; then exit 0; fi; sleep 2;  done;
          env:
            - name: KONG_PG_HOST
              value: postgres2
            - name: KONG_PG_PASSWORD
              value: kong
          image: hub.eos.h3c.com/kong/kong:2.2
          name: wait-for-migrations
      serviceAccountName: kong-serviceaccount2
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres2
  namespace: kong2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres2
  serviceName: postgres2
  template:
    metadata:
      labels:
        app: postgres2
    spec:
      #      nodeSelector:
      #        kubernetes.io/hostname: k8s-04
      containers:
        - env:
            - name: POSTGRES_USER
              value: kong
            - name: POSTGRES_PASSWORD
              value: kong
            - name: POSTGRES_DB
              value: kong
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          image: hub.eos.h3c.com/base/postgres:9.5
          name: postgres
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: datadirpg
              subPath: pgdata
              readOnly: false
      volumes:
        - name: datadirpg
          emptyDir: {}
      terminationGracePeriodSeconds: 60
  # volumeClaimTemplates:                   # 用于持久化的模板
  #   - metadata:
  #       name: datadirpg
  #       annotations:
  #         volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
  #     spec:
  #       accessModes:
  #         - ReadWriteOnce
  #       resources:
  #         requests:
  #           storage: 1Gi
  # volumeClaimTemplates:
  #   - metadata:
  #       name: datadir
  #     spec:
  #       accessModes: ["ReadWriteOnce"]
  #       storageClassName: "eos-ceph"
  #       resources:
  #         requests:
  #           storage: 1Gi
#  volumeClaimTemplates:
#    - metadata:
#        name: datadir
#      spec:
#        accessModes:
#          - ReadWriteOnce
#        resources:
#          requests:
#            storage: 1Gi
---
apiVersion: batch/v1
kind: Job
metadata:
  name: kong-migrations2
  namespace: kong2
spec:
  template:
    metadata:
      name: kong-migrations2
    spec:
      containers:
        - command:
            - /bin/sh
            - -c
            - kong migrations bootstrap
          env:
            - name: KONG_PG_PASSWORD
              value: kong
            - name: KONG_PG_HOST
              value: postgres2
            - name: KONG_PG_PORT
              value: "5432"
          image: hub.eos.h3c.com/kong/kong:2.2
          name: kong-migrations
      initContainers:
        - command:
            - /bin/sh
            - -c
            - until nc -zv $KONG_PG_HOST $KONG_PG_PORT -w1; do echo 'waiting for db'; sleep 1; done
          env:
            - name: KONG_PG_HOST
              value: postgres2
            - name: KONG_PG_PORT
              value: "5432"
          image: hub.eos.h3c.com/kong/busybox
          name: wait-for-postgres
      restartPolicy: OnFailure
```

konga 部署

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kong-konga
  namespace: kong2
spec:
  selector:
    matchLabels:
      app: kong-konga
  replicas: 1
  template:
    metadata:
      labels:
        app: kong-konga
    spec:
      #inodeSelector:
      # node: worker
      containers:
        - name: kong-konga
          image: hub.eos.h3c.com/kong/konga:next
          imagePullPolicy: IfNotPresent
          env:
            - name: DB_ADAPTER
              value: postgres
            - name: DB_HOST
              value: postgres2
            - name: DB_PORT
              value: "5432"
            - name: DB_USER
              value: kong
            - name: DB_DATABASE
              value: konga
            - name: DB_PASSWORD
              value: kong
            - name: NODE_ENV
              #value: production
              value: development
            - name: TZ
              value: Asia/Shanghai
          ports:
            - containerPort: 1337
---
apiVersion: v1
kind: Service
metadata:
  name: kong-konga
  namespace: kong2
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 1337
  #      nodePort: 31337
  type: NodePort
  selector:
    app: kong-konga
---
```

KongIngress

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongIngress
metadata:
  name: kong-lee-override
  namespace: kong2
route:
  strip_path: true
# proxy:
#   path: /
```

nginx 测试

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: kong2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-pod
  template:
    metadata:
      labels:
        app: myapp-pod
    spec:
      containers:
      - name: myapp
        image: hub.eos-ts.h3c.com/nginx:1.19.3-alpine
        ports:
        - containerPort: 80
          name: web
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: kong2
  # annotations:
  #   konghq.com/override: "kong2"
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: myapp-pod
  type: NodePort
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: kong2
  annotations:
    kubernetes.io/ingress.class: "kong-lee"
    konghq.com/strip-path: "true"
spec:
  rules:
  - host: myapp.h3c.com
    http:
      paths:
      - path: /nginx
        backend:
          serviceName: myapp
          servicePort: 80
      - path: /xxx
        backend:
          serviceName: myapp
          servicePort: 80
```
