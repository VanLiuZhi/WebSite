---
weight: 1
title: "K8s Istio"
subtitle: "K8s Istio"
date: 2021-01-07T10:17:32+08:00
lastmod: 2021-01-07T10:17:32+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Istio"
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



<!--more-->

## 流量管理

虚拟服务（Virtual Service） 和目标规则（Destination Rule） 是 Istio 流量路由功能的关键拼图

Istio 维护了一个内部服务注册表 (service registry)，它包含在服务网格中运行的一组服务及其相应的服务 endpoints。Istio 使用服务注册表生成 Envoy 配置

## 安全配置

```s
kubectl apply -f - <<EOF
apiVersion: "security.istio.io/v1beta1"
kind: "RequestAuthentication"
metadata:
  name: "jwt-example"
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  jwtRules:
  - issuer: "liuzhi@h3c.com"
    jwks: |
      {
          "keys": [
            {
              "e":"AQAB",
              "kid":"YTThkTiDINDMj3Ye0hltY9-Q0UAYoUp1Fqb9nyMtWJU",
              "kty":"RSA","n":"rc_hC4lQJ0zpw6-Q9ENooyAOjB1aUiB_W5YYYU8oS2vuTX20GTYT1ud0Jr1QXya4nF4FgxMyDqQDHQ6ON_naJyXvHWFleqNfu_Vm7lXAI_vB0NT3RiU4z9-vwyZTbOcEAixYYbJr2nLhE_jjiD3C2D57BE3A6qmYfCQGMG1RgkBHWWJXbM8I1FdEerv3kgVD1SCeKDUEka60lIYb6J-k3n7fy9q3ktfOz3Plgvb3DUJHaYDY9J2umlxUBuk8nz088IYfRxetZlNQV0Ovm-3m_f_ojnpv03IuldVsueIqxXWtYOnpa8fRgWcRTWXTrTivaVjlTMn1Q4PdBiiEkqcKpw"
            }
          ]
      }
EOF
```

kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl http://httpbin.foo:8000/ip -s -o /dev/null -w "%{http_code}\n"


kubectl exec "$(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name})" -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"

export TOKEN=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJsaXV6aGlAaDNjLmNvbSIsInN1YiI6ImxpdXpoaUBoM2MuY29tIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjQ2ODU5Nzk4MDB9.FJmG_MXgM1eK17UhDcKUHU6J7D6FmCiTbaH2fk0D_GYf3CRqzb4Z0Fgw-lIJWtzPrlh_mNCSDHXZac5mmUNv42dNAm9d8RL1MxgZwbQW8Ur1_SidsI0gSu3Go5gllSG1k_YUSrx8DbCDtn_U5F_Pgg8CdIXkRIiez8ZjuEHpW9gGjp4StlSj4lwd3pLeu_DDXSlzZDPoc9cH1I_A1Qm_E5SaU-g3jMJCJ98tT-lXRmhQyqF1Qvz_F37zom1I_GRDozeKXMeMmuzy1PXP3JAhj2xCw5cCbR_7l_pTwCGWrbnJ7XtkpXkVH0tlDQYF3pJV94AJux6SNMp1H2tOdf3vWA

export TOKEN=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJsaXV6aGlAaDNjLmNvbSIsInN1YiI6ImxpdXpoaUBoM2MuY29tIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjQ2ODU5Nzk4MDB9.FJmG_MXgM1eK17UhDcKUHU6J7D6FmCiTbaH2fk0D_GYf3CRqzb4Z0Fgw-lIJWtzPrlh_mNCSDHXZac5mmUNv42dNAm9d8RL1MxgZwbQW8Ur1_SidsI0gSu3Go5gllSG1k_YUSrx8DbCDtn_U5F_Pgg8CdIXkRIiez8ZjuEHpW9gGjp4StlSj4lwd3pLeu_DDXSlzZDPoc9cH1I_A1Qm_E5SaU-g3jMJCJ98tT-lXRmhQyqF1Qvz_F37zom1I_GRDozeKXMeMmuzy1PXP3JAhj2xCw5cCbR_7l_pTwCGWrbnJ7XtkpXkVH0tlDQYF3pJV94AJux6SNMp1H2tOdf3vWA

export TOKEN=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJsaXV6aGlAaDNjLmNvbSIsInN1YiI6ImxpdXpoaUBoM2MuY29tIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjQ2ODU5Nzk4MDAsImdyb3VwcyI6WyJncm91cDEiLCJlb3MtYWRtaW4iXX0.eCRSYSPA5cR8OPXWfxKRWD2pr0uH2QadaKDT1EGMaTq8WyX_hKr09fpuJAosevhWlc10crg4qRN-ylXNSgkXxTT1WaKVphijIL1SXO-AY39UfSAljEpAU-ZHqrjss6L0wjyoA_rp6wrRQM-GtrA_U-hn2rxP9tXG5425eC1Ixsdb7suVYTHQvXSQEBHZLS9gjauPOA8-Ah47MsovGzg4MBOeks-PTER52LgW1hBWSDohyJpC0nNvllDGwVNaLRfx0ofKxll7WEFVyZv5ctAOphAfVDLfGijxoAeOzTya8gvgi2J6Au3zkSVN19fA4Prb4OUXmwVIBlcDW3jdb4qnqA

export TOKEN=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJsaXV6aGlAaDNjLmNvbSIsInN1YiI6ImxpdXpoaUBoM2MuY29tIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjQ2ODU5Nzk4MDAsImdyb3VwcyI6WyJncm91cDEiXX0.Hc4s8BUTasjo1pyllUr2SWOVPjKjcVjBzkssJuL45dEcYXoH4NaKTMdu0lVsbEycBCfNjlGlzvk39O-6_vpt1qDjQwxL-DCFpenZcU8Ib5InLT279jS7pnYU28Lf2X_ZVFv6pDTElYwLlufBEHBGbGRR80FkmNuISwHU4LeMIca-tM-5bgYCXHf_5ALU4ujEP2GpV8P6_8o1_aIqU9kQB1Y7Jqqt45AkEjx0w0L4pBt6UzsopLZf3AX0IldOR8ryhf8PHlXrygyXnvu9XCwFNunJM0vTwWknLv46UMls7U6xNLlWx6Bt4ZPSmjCiCjm1pOjehH00Xuw1yDKRAGM-IQ

kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"



kubectl delete -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["liuzhi@h3c.com/liuzhi@h3c.com"]
EOF

$ kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["liuzhi@h3c.com/liuzhi@h3c.com"]
    when:
    - key: request.auth.claims[groups]
      values: ["group1"]
EOF


kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["liuzhi@h3c.com/liuzhi@h3c.com"]
    when:
    - key: request.auth.claims[groups]
      values: ["eos-admin"]
EOF


前言
云原生的应用为组织异构语言的系统提供了可能，我们可以把Java，Python等拆成微服务，利用容器的特性屏蔽环境等问题，但是随着微服务数量的增多，也带来了更多的服务治理的问题

类似限流熔断等能力，对于Java有成熟的解决方案，但是python就不一定了，可能需要我们自己造轮子。

类似的问题还有鉴权体系，对于企业级开发语言Java来说，有开箱即用方案，其它语言就很可能需要自己造轮子了



而Service Mesh可以解决这一系列问题，把这种通用的基础设施下沉，让开发者只关注业务代码，本文将讲解 istio 基于JWT授权的方式

准备环境
Kubernetes 1.18.0  istio 1.7.4

保证各个组件正常，我们创建官方的测试例子，这会创建两个服务，它们本身是没有任何鉴权体系的，通过istio控制平面把配置注入到代理中，实现基于JWT的授权

$ kubectl create ns foo

$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo

$ kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo



准备JWK

JWK：JWT的密钥，也就是我们常说的 scret；我们先生成一个



# 需要先安装依赖: pip3 install jwcrypto -i https://pypi.tuna.tsinghua.edu.cn/simple
from pathlib import Path

from jwcrypto.jwk import JWK

private_key = Path("rsa-private-key.pem").read_bytes()
jwk = JWK.from_pem(private_key)

# 导出公钥 RSA Public Key
public_key = jwk.public().export_to_pem()
with open('./public.pem', 'w') as f: # 设置文件对象
f.write(bytes.decode(public_key))

print("=" * 30)

# 导出 JWK
jwk_bytes = jwk.public().export()
with open('./jwt.json', 'w') as f: # 设置文件对象
f.write(jwk_bytes)

通过python代码，生成需要的文件





创建RequestAuthentication
把jwt.json的配置写到jwks中



kubectl apply -f - <<EOF
apiVersion: "security.istio.io/v1beta1"
kind: "RequestAuthentication"
metadata:
  name: "jwt-example"
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  jwtRules:
  - issuer: "liuzhi@h3c.com"
    jwks: |
      {
          "keys": [
            {
              "e":"AQAB",
              "kid":"YTThkTiDINDMj3Ye0hltY9-Q0UAYoUp1Fqb9nyMtWJU",
              "kty":"RSA","n":"rc_hC4lQJ0zpw6-Q9ENooyAOjB1aUiB_W5YYYU8oS2vuTX20GTYT1ud0Jr1QXya4nF4FgxMyDqQDHQ6ON_naJyXvHWFleqNfu_Vm7lXAI_vB0NT3RiU4z9-vwyZTbOcEAixYYbJr2nLhE_jjiD3C2D57BE3A6qmYfCQGMG1RgkBHWWJXbM8I1FdEerv3kgVD1SCeKDUEka60lIYb6J-k3n7fy9q3ktfOz3Plgvb3DUJHaYDY9J2umlxUBuk8nz088IYfRxetZlNQV0Ovm-3m_f_ojnpv03IuldVsueIqxXWtYOnpa8fRgWcRTWXTrTivaVjlTMn1Q4PdBiiEkqcKpw"
            }
          ]
      }
EOF



这样我们的httpbin就被保护起来的，需要传递token



打开  https://jwt.io/#debugger-io  把我们生成的密钥填入，生成一个token







使用错误的token访问，code 401

kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"



使用正确签名的token，code 200

kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"



不过这还没配置完成，istio只是校验token是否正确，如果不传递token访问也是200的

配置 AuthorizationPolicy


通过配置AuthorizationPolicy来解决这个问题



kubectl delete -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["liuzhi@h3c.com/liuzhi@h3c.com"]
EOF



对httpbin的应用执行ALLOW的动作，前提是规则匹配正确



使用正确签名的token，code 200

kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"



不传递token，code 403

kubectl exec $(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name}) -c sleep -n foo -- curl "http://httpbin.foo:8000/headers" -s -o /dev/null -w "%{http_code}\n"



如此，我们就实现了通过istio为微服务增加鉴权保护



当然这个配置还是不够灵活，我们可以对jwt增加自定义的载荷，修改AuthorizationPolicy的策略。比如增加"groups":["group1，eos-admin"]

可以测试一下，只有带eos-admin载荷的jwt才能访问接口



kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["liuzhi@h3c.com/liuzhi@h3c.com"]
    when:
    - key: request.auth.claims[groups]
      values: ["eos-admin"]
EOF

总结
以上就是Service Meash架构下，通过istio解决服务鉴权的例子，不过这个示例还是过于简单，对于生产来说肯定是不适用的

存在的问题：

所有的权限信息要配置在jwt中
配置yaml策略过于繁琐
如何组织jwt载荷的数据结构
生产的运用还是不能像这种简单的示例来做的，需要我们来扩展实现符合自身的鉴权体系



比如 keycloak+istio实现基于jwt的服务认证授权

keycloak提供了UI的配置页面，使用keycloak结合istio可以实现细粒度的认证授权策略，客户端只需要到认证授权中心获取token，服务端无需关心任何认证授权细节，专注以业务实现，实现业务逻辑与基础设施的解耦

参考
https://blog.csdn.net/yin0501/article/details/109319901

https://cloud.tencent.com/developer/article/1697063

https://jwt.io/#debugger-io

https://developer.51cto.com/art/202011/630971.htm

https://juejin.cn/post/6899322135924178952

