---
weight: 1000
title: "JavaScriptUtil 常用实现"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "JavaScriptUtil 常用实现"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Util, JavaScript]
categories: [Util]

lightgallery: true

toc:
  auto: false
---

常见的JavaScript 相关设计

<!-- more -->

## 自定义遮蔽罩

使用了jQuery-WeUI，需要根据情况做调整

```html

<!--自定义遮罩层-->
<div id="bg" class="weui-mask weui-mask--visible"
    style="display: none;opacity: 1;visibility: visible;z-index: 100">
</div>

<!-- 简单示例 -->
<!DOCTYPE html>
<html>

<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <title>html 最简遮罩层</title>
    <script type="text/javascript">
        function showDiv() {
            document.getElementById('popDiv').style.display = 'block';
            document.getElementById('bg').style.display = 'block';
        }

        function closeDiv() {
            document.getElementById('popDiv').style.display = 'none';
            document.getElementById('bg').style.display = 'none';
        }
    </script>
</head>

<body>
<div id="popDiv"
    style="z-index:99;display:none;position:absolute;margin-top: 20%;margin-left: 40%;background-color: #FFF;">html
    最简遮罩层<br/>html 最简遮罩层<br/>
    <a href="javascript:closeDiv()">关闭遮罩层</a>
</div>
<div id="bg"
    style="display:none;background-color: #ccc;width: 100%;position:absolute;height: 100%;opacity: 0.5;z-index: 1;"></div>
<div style="padding-top: 10%;padding-left:40%;z-index:1;">
    <input type="Submit" name="" value="打开遮罩层" onclick="javascript:showDiv()"/>
</div>
</body>

</html>

```