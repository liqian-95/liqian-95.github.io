---
layout:     post
title:      "serverless"
subtitle:   "无服务架构"
date:       2018-12-10
author:     "Qian"
# header-img: "img/in-post/post-bg-tensorflow.jpg"
tags:
    - edge
---


## Serverless概念

传统的互联应用架构图与serverless架构的主要不同点在于Serverless架构能够让开发者在构建应用的过程中无需关注计算资源的获取和运维，由平台来按需分配计算资源并保证应用执行的SLA，按照调用次数进行计费，有效的节省应用成本。传统的互联网APP主要采用C/S架构，服务器端需长期维持业务进程来处理客户端请求，并调用代码逻辑完成请求响应流程。而在Serverless架构中，应用业务逻辑将基于FAAS架构形成独立为多个相互独立功能组件，并以API服务的形式向外提供服务，业务代码仅在调用时才激活运行，当响应结束占用资源便会释放。

Serverless是最新兴起的架构模式，中文意思是“无服务器”架构。目前，业界并没有给出明确的定义，把其分成两种类型，分别为“Backend as a Service” 和 “Functions as a Service”。

“Backend as a Service”即BaaS，是一种新型的云服务，旨在为移动和Web应用提供后端云服务，实现对逻辑和状态进行管理，包括云端数据/文件存储（例如Parse、Firebase）、消息推送（例如极光推送、个推）、应用数据分析等等。

“Functions as a Service”即FaaS，指这样的应用，一部分服务逻辑由应用实现，但是跟传统架构不同在于，他们运行于无状态的容器中，可以由事件触发，短暂的，完全被第三方管理，功能上FaaS就是不需要关心后台服务器或者应用服务，只需关心自己的代码即可。
