---
layout: post
title: "蚂蚁金服 Service Mesh 双十一实战"
description: "当 Service Mesh 遇到双十一又会迸发出怎样的火花？蚂蚁金服的 LDC 架构继续演进的过程中，Service Mesh 要承载起哪方面的责任？我们借助四个“双十一考题”一一为大家揭晓。"
category: service-mesh 
tags: [service-mesh, golang]
---

> 2019 年的双十一是蚂蚁金服的重要时刻，大规模落地了 Service Mesh 并顺利保障双十一平稳渡过。我们第一时间与这次的落地负责人进行了交流。
> 
> 采访的开头：
> 花肉：“这次大规模上了 Service Mesh ，双十一值班感觉是什么？”
> 卓与：“Service Mesh 真的稳。”

{: refdef: style="text-align: center;"}
![卓与 TOP100 北京峰会分享现场图](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558287-796b7ea3-a30d-4ff8-b020-c41ae8bb1dbb.jpeg)

图为卓与 TOP100 北京峰会分享现场图

## 落地负责人介绍

Service Mesh 是蚂蚁金服下一代架构的核心，今年蚂蚁金服大规模的 Service Mesh 落地，我有幸带领并面对了这个挑战，并非常平稳的通过了双十一的大考。

我个人主要专注在微服务领域，在服务注册与服务框架方向深耕多年，主导过第五代服务注册中心（SOFARegistry）设计与实施，在微服务的架构演进中持续探索新方向，并在蚂蚁金服第五代架构演进中负责内部 Service Mesh 方向的架构设计与落地。

* SOFAStack：[https://github.com/sofastack](https://github.com/sofastack)
* SOFAMosn：[https://github.com/sofastack/sofa-mosn](https://github.com/sofastack/sofa-mosn)

## Service Mesh 在蚂蚁金服

蚂蚁金服很早开始关注 Service Mesh，并在 2018 年发起 ServiceMesher 社区，目前已有 4000+ 开发者在社区活跃。在技术应用层面，Service Mesh 的场景已经渡过探索期，今年已经全面进入深水区探索。

2019 年的双十一是我们的重要时刻，我们进行了大规模的落地，可能是目前业界最大规模的实践。作为技术人能面对世界级的流量挑战，是非常紧张和兴奋的。当 Service Mesh 遇到双十一又会迸发出怎样的火花？蚂蚁金服的 LDC 架构继续演进的过程中，Service Mesh 要承载起哪方面的责任？我们借助四个“双十一考题”一一为大家揭晓。

## Service Mesh 背景知识

Service Mesh 这个概念社区已经火了很久，相关的背景知识从我们的公众号内可以找到非常多的文章，我在这里不做过于冗余的介绍，仅把几个核心概念统一，便于后续理解。

{: refdef: style="text-align: center;"}
![图1. Service Mesh 开源架构来自 [https://istio.io/](https://istio.io/)](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558293-58c065c9-c3f2-4d0b-ab23-72566052344e.png)

图1. Service Mesh 开源架构来自 [https://istio.io/](https://istio.io/)

Istio 的架构图上清晰的描述了 Service Mesh 最核心的两个概念：数据面与控制面。数据面负责做网络代理，在服务请求的链路上做一层拦截与转发，可以在链路中做服务路由、链路加密、服务鉴权等，控制面负责做服务发现、服务路由管理、请求度量（放在控制面颇受争议）等。

Service Mesh 带来的好处不再赘述，我们来看下蚂蚁金服的数据面和控制面产品，如下图：

{: refdef: style="text-align: center;"}
![图2. 蚂蚁金服 Service Mesh 示意架构](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558297-1ee1945c-3903-4695-add9-76d5d9febf34.png)

图2. 蚂蚁金服 Service Mesh 示意架构

**数据面：SOFAMosn。**蚂蚁金服使用 Golang 研发的高性能网络代理，作为 Service Mesh 的数据面，承载了蚂蚁金服双十一海量的核心应用流量。

**控制面：SOFAMesh。**Istio 改造版，落地过程中精简为 Pilot 和 Citadel，Mixer 直接集成在数据面中避免多一跳的开销。

## 2019 Service Mesh 双十一大考揭秘

双十一 SOFAMosn 与 SOFAMesh 经历海量规模大考，顺利保障双十一平稳渡过。今年双十一蚂蚁金服的百十多个核心应用全面接入 SOFAMosn，生产 Mesh 化容器几十万台，双十一峰值 SOFAMosn 承载数据规模数千万 QPS，SOFAMosn 转发平均处理耗时 0.2ms。

{: refdef: style="text-align: center;"}
![图3. 双十一落地数据](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558305-41757a40-9b46-46a7-8967-d2de6dae78a8.png)

图3. 双十一落地数据

在如此大规模的接入场景下，我们面对的是极端复杂的场景，同时需要多方全力合作，更要保障数据面的性能稳定性满足大促诉求，整个过程极具挑战。下面我们将从几个方面来分享下我们在这个历程中遇到的问题及解决方案。

## 双十一考题

1. 如何让 Service Mesh 发挥最大的业务价值？
1. 如何达成几十万容器接入 SOFAMosn 的目标？
1. 如何处理几十万容器 SOFAMosn 的版本升级问题？
1. 如何保障 Service Mesh 的性能与稳定性达标？

## 落地架构

为了更加方便的理解以上问题的解决与后续介绍中可能涉及的术语等，我们先来看下 Service Mesh 落地的主要架构：

{: refdef: style="text-align: center;"}
![图4. Service Mesh 落地架构](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558302-93bd5a33-ce66-46ed-9162-fd75d4fb27d1.png)

图4. Service Mesh 落地架构

以上架构图中主要分几部分：

1. 数据面：借助 Kubernetes 中的 Pod 模型，SOFAMosn 以独立镜像和 App 镜像共同编排在同一个 Pod 内，共享相同的 Network Namespace、CPU、Memory，接入 SOFAMosn 后所有的 App RPC 流量、消息流量均不在直接对外，而是直接和 SOFAMosn 交互，由 SOFAMosn 直接对接服务注册中心做服务发现，对接 Pilot 做配置下发，对接 MQ Server 做消息收发等；
1. 控制面：由 Pilot、Citadel 和服务注册中心等组件组成，负责服务地址下发、服务路由下发、证书下发等；
1. 底层支撑：Sidecar 的接入与升级等均依赖 Kubernetes 能力，通过 webhook 做 Sidecar 的注入，通过 Operator 做 Sidecar 的版本升级等，相关运维动作均离不开底层的支撑；
1. 产品层：结合底层提供的原子能力做运维能力封装，监控对接，采集 Sidecar 暴露的 Metrics 数据做监控与预警，流量调控，安全等产品化能力；

## 蚂蚁金服的答卷

### 1. 如何让 Service Mesh 发挥最大的业务价值？

作为一个技术人，我们非常坚持不要为了技术革新本身去做技术革新，**一定要让技术帮助业务，用技术驱动业务**。这几年随着大家消费习惯以及网络行为的改变，双十一面对的业务场景远比想象中复杂。举个例子大家感受下，大家有没有发现你的女友或者老婆每天对着李佳琦的淘宝直播购物，主播们不断的、实时的红包、上新等等，带来了远比秒杀更复杂的业务场景和体量。大促的模式也更加丰富，不同场景下的大促涉及的应用是不同的，每一类应用在应对独特的洪峰时都需要有足够的资源。

假如运营同学在不同时间点设计了两种活动，两种活动分别会对应两类不同的应用，如果这两类应用都在大促前准备充足的资源自然是轻松渡过大促峰值，但大促洪峰时间短暂，大量的资源投入有很长一段时间都处于空闲状态，这自然不符合技术人的追求。

那么如何在不增加资源的场景下渡过各种大促呢？

核心问题就是如何在足够短的时间内做到大规模资源腾挪，让一批机器资源可以在不同时间点承载起不同的大促洪峰。

面对这样的挑战，我们会有怎样的解法呢？

Service Mesh 618 大促落地试点时，我们有介绍到为什么要做这个事情，核心价值是业务与基础设施解耦，双方可以并行发展，快速往前走。那么并行发展究竟能为业务带来哪些实际的价值？除了降低基础组件的升级难度之外，我们还在资源调度方向做了以下探索：

#### 1.1 腾挪调度

说到资源调度，最简单的自然是直接做资源腾挪，比如大促A峰值过后，大促A对应的应用通过缩容把资源释放出来，大促B对应的应用再去申请资源做扩容，这种方式非常简单，难点在于当资源规模非常庞大时，缩容扩容的效率极低，应用需要把资源申请出来并启动应用，耗时很长，假如多个大促之间的时间间隔很短，腾挪需要的耗时极有可能超过大促时间间隔，这种调度方案已不能满足双十一多种大促的诉求。

{: refdef: style="text-align: center;"}
![图5. 腾挪调度](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558298-65f6f60b-b019-4e1c-abca-6757f39732ba.png)

图5. 腾挪调度

#### 1.2 分时调度

腾挪调度最大的问题是应用需要冷启动，冷启动本身的耗时，加上冷启动后预热需要的耗时，对整个腾挪时间影响极大。假如应用不需要启动就能完成资源腾挪，整个调度速度将会加快很多。基于应用不需要重新启动即可完成资源腾挪的思路，我们提出了分时调度的方案：分时调度会通过超卖的机制申请出足够的容器。

{: refdef: style="text-align: center;"}
![图6. 分时调度](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558307-7fe7df2e-8f88-40fa-97fb-06eb63a732c6.png)

图6. 分时调度

我们把资源池中的应用容器分为两种状态（4C8G 规格为例）：

- **运行态：**运行态的应用处于全速运行的状态（资源可使用到 4C8G），它们可以使用充足的资源全速运行，承载 100% 的流量；
- **保活态：**保活态的应用处于低速运行的状态（资源可使用到 1C2G），它们仅可使用受限的资源低速运行，承载 1% 的流量，剩余 99% 的流量由 SOFAMosn 转发给运行态节点；

{: refdef: style="text-align: center;"}
![图7. 保活态流量转发](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558347-836c91a2-9b43-4f35-be54-ec2547b230c9.png)

图7. 保活态流量转发

保活态和运行态可以快速切换，切换时仅需要做 JVM 内存 Swap 以及基于 SOFAMosn 的流量比例切换，即可快速做到大促A到大促B间资源的快速切换。

SOFAMosn 流量比例切换的能力是分时调度中非常核心的一部分，有了这份流量转发，Java 内存 Swap 后才可以降低流量以保持各种连接活性，如应用与 DB 间的连接活性等。通过分时调度的技术手段，我们达成了双十一多种大促不加机器的目标，这个目标达成后能节省的成本是极大的，这也是 Service Mesh 化后将能带来的重要业务价值。

每个公司落地 Service Mesh 的路径不尽相同，这里给出的建议依然是不要为了技术革新本身去做技术革新，多挖掘 Service Mesh 化后能给当前业务带来的实际价值，并以此价值为抓手设定目标，逐步落地。

### 2. 如何达成几十万容器接入 SOFAMosn 的目标？

说到数据面的接入，了解社区方案的同学应该会联想到 Istio 的 Sidecar Injector。Istio 通过在 Kubernetes 中集成 Sidecar Injector 来注入 Envoy。标准的 Kubernetes 更新行为也是 Pod 的重建，创建新 Pod 销毁旧 Pod，更新过程中 IP 会改变等。

在蚂蚁金服的场景下，或者说在国内较多公司使用 Kubernetes 做内部运维体系升级时，都会走先替换底层资源调度再替换上层 Paas 产品的路子，因为已有的 Paas 是客观存在的，Paas 的能力也非一朝一夕可以直接云原生化的，那么先用 Kubernetes 替换掉底层的资源调度的路子就变的顺理成章了。这条路子上还涉及非 Kubernetes 体系和 Kubernetes 体系的兼容性，如保持 IP 地址不变的 Pod 更新能力，有了这些能力对上层 Paas 来讲，运维体系和运维传统 VM 的场景类似，只是发布部署可以由上传 zip 包解压发布变成镜像化发布，同时在更多高阶场景可以借助 Operator 实现更加面向终态的运维模式。

{: refdef: style="text-align: center;"}
![图4. Service Mesh 落地架构](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558328-84761913-5c31-46df-90e6-3b41b5cffeca.png)

我们上面介绍 图4. Service Mesh 落地架构 时，有看到 SOFAMosn 和应用 App 编排在同一个 Pod 内，那么一个应用要接入 SOFAMosn 首先要先完成镜像化，可以被 Kubernetes 管理起来，然后是 SOFAMosn 的注入，注入后还需要可原地更新（InPlace Update）。

#### 2.1 替换接入

在我们落地的初期，SOFAMosn 的接入方式主要是替换接入。替换接入就是指定某个应用做 SOFAMosn 接入时，我们会通过创建新 Pod，然后在创建的同时注入 SOFAMosn，新 Pod 创建完成后缩容旧容器。为什么说是缩容旧容器而不是说缩容旧 Pod，是因为接入早期，我们内部的调度系统 Sigma（蚂蚁金服版 Kubernetes）还处于底层逐步替换的过程中，有部分容器还是非 Kubernetes 管理的状态，这部分容器没有 Pod 的概念。

{: refdef: style="text-align: center;"}
![图8. 替换接入](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558322-504de535-ab29-45d2-849f-0edefa3f2abd.png)

图8. 替换接入

替换接入最大的问题就是需要充足的资源 Buffer，否则新的 Pod 调度不出来，接入进度便会受阻，而且整个接入周期加上资源申请，以及资源申请出后附带的一系列周边配套系统的数据初始化等，周期较长、依赖较多，所以这个能力是很难在短时间内满足我们几十万容器的接入进度目标的。

#### 2.2 原地接入

基于替换接入的已知问题，我们尝试结合资源调度层的改造，将接入方案做了大幅简化，并提供了原地接入能力。原地接入会首先把不在 Kubernetes 集群中管理的容器原地替换为由 Kubernetes 集群管理的 Pod，然后通过 Pod Upgrade 时直接新增 Sidecar 的方式将 SOFAMosn 注入 Pod，省去了扩容的资源 Buffer 开销，将整个接入过程变的简单顺滑。

{: refdef: style="text-align: center;"}
![图9. 原地接入](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558359-b940f241-6ebd-4062-a767-6f6d0b8e8286.png)

图9. 原地接入

原地接入打破了仅在创建 Pod 时注入的原则，带来了更高的复杂度。如原 Pod 拥有 4C8G 的资源，注入 SOFAMosn 后不能新增资源占用，如果新增了资源占用那么这部分资源就不受容量管控系统管理，所以我们将 App Container 和 SOFAMosn Container 的资源做了共享，SOFAMosn 和 APP 公用 4C 的 CPU 额度，抢占比相同。内存方面 SOFAMosn 可用内存 Limit 为 App 的 1/4，对于 8G 内存来讲，SOFAMosn 可见 2G，应用依然可见 8G，当应用使用内存超过 6G 时，由 OOM-Killer 优先 kill SOFAMosn 来保障应用可继续申请到内存资源。

以上 CPU 抢占比和 Mem limit 比例均为实际压测调整比例，最初我们尝试过 SOFAMosn CPU 占比为应用的 1/4，Mem 为应用的 1/16，这在小流量的应用下可正常工作，但是遇到核心应用，连接数上万的场景下，内存和 CPU 都比较吃紧，特别是 CPU 争抢能力弱于应用的场景下，会导致 RT 变长，间接影响应用。在实际运行的稳定性保障上通过各种参数调整和验证，最后得出 SOFAMosn 与 APP 1:1 共享 CPU，Mem limit 限制为应用 Mem 1/4 的比例。

{: refdef: style="text-align: center;"}
![图10. 空中加油](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558337-10999629-f3a1-4332-aa5d-dac5685a0906.png)

图10. 空中加油

通过原地注入的能力，我们实现了更加平滑的接入方式，极大加速了海量容器接入 SOFAMosn 的进程。

### 3. 如何处理几十万容器 SOFAMosn 的版本升级问题？

海量容器接入 SOFAMosn 之后，一个更大的挑战是如何快速的升级 SOFAMosn 版本。我们一直强调 Service Mesh 带来的核心价值是业务与基础设施层的解耦，让双方可以更加快捷的向前走，但假如没有快速升级的能力，那么这个快速往前走便成了一句空话，而且任何软件都可能引入 Bug，我们也需要一个快速的机制在 Bug 修复时可以更加快速的升级或回滚。SOFAMosn 的升级能力也从有感升级进阶到无感升级。

#### 3.1 有感升级

{: refdef: style="text-align: center;"}
![图11. 有感升级](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558337-00a17560-8ae0-4acf-9fd3-cdc169b31111.png)

图11. 有感升级

有感升级是一次 Pod 的 InPlace Update，这个升级过程中，会关闭此 Pod 的流量，然后做容器 Upgrade，容器 Upgrade 的时候，应用和 SOFAMosn 均会做更新、启动，并在升级完成后打开流量。这个过程的耗时主要取决于应用启动耗时，如一些庞大的应用启动耗时可能达 2~5分钟之久，所以升级周期也会被拉长，并且应用能感知到流量的开关。所以这种方式我们称之为有感升级，这种方式的优点是升级过程中无流量，稳定性更高。

#### 3.2 无感升级

{: refdef: style="text-align: center;"}
![图12. 无感升级](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558351-fa702c0e-126a-43c2-88c2-2a8e65ad8c4e.png)

图12. 无感升级

无感升级我们又称之为平滑升级，整个升级过程中不会暂停流量，SOFAMosn 的版本直接通过热更新技术，动态接管旧版本的流量，并完成升级，整个过程中 APP Container 无感知，所以可以称为无感升级。这种升级方式减少了应用重启的耗时，整个升级时长消耗主要在新旧 SOFAMosn 版本之间的连接迁移，关于无感升级的实现细节，可以参考我 2019 年上半年在 GIAC 上的分享：[《蚂蚁金服 Service Mesh 落地实践与挑战 | GIAC 实录》](https://mp.weixin.qq.com/s/lo23CB0Ibp2VEkBT_VgPsg)

#### 3.3 无人值守

有了快速升级的能力，还需要有在升级过程中的风险拦截，这样才能放心大胆的做版本升级。基于无人值守的理念，我们在 SOFAMosn 的升级前后基于 Metrics 度量指标判断升级前后的流量变化、成功率变化以及业务指标的变化等，实现及时的变更风险识别与阻断，通过这种方式真正做到了放心大胆的快速升级。

{: refdef: style="text-align: center;"}
![图13. 无人值守](https://cdn.jsdelivr.net/gh/jervyshi/jervyshi.github.io/assets/images/1574056558350-0bb7168a-e244-4932-ac3c-b1b5c75e3904.png)

图13. 无人值守

### 4. 如何保障 Service Mesh 的性能与稳定性达标？

SOFAMosn 在双十一的表现稳定，峰值千万级 QPS 经过 SOFAMosn，SOFAMosn 内部处理消耗平均 RT 低于 0.2ms。我们在性能和稳定性上经过多重优化改进才最终能达成以上成果，这其中离不开生产的持续压测、打磨，以下是我们在性能和稳定性方面的一些改进点，仅供参考：

#### 4.1 性能优化

**CPU 优化**

Golang writev 优化：多个包拼装一次写，降低 syscall 调用。我们在 go 1.9 的时候发现 writev 有内存泄露的bug，内部使用的目前是 patch 了 writev bugfix 的 go 1.12。writev bugfix 已经集成在 go 1.13 中。

详情见我们当时给go 提交的PR： [https://github.com/golang/go/pull/32138](https://github.com/golang/go/pull/32138)

**内存优化**

内存复用：报文直接解析可能会产生大量临时对象，SOFAMosn 通过直接复用报文字节的方式，将必要的信息直接通过 unsafe.Pointer 指向报文的指定位置来避免临时对象的产生。

**延迟优化**

协议升级：快速读取 header：TR 协议请求头和 Body 均为 hessian 序列化，性能损耗较大。而 Bolt 协议中 Header 是一个扁平化map，解析性能损耗小。升级应用走 Bolt 协议来提升性能。

路由缓存：内部路由的复杂性（一笔请求经常需要走多种路由策略最终确定路由结果目标），通过对相同条件的路由结果做秒级缓存，我们成功将某核心链路的全链路 RT 降低 7%。

#### 4.2 稳定性建设

- Pod 级别 CPU Mem 限额配置，Sidecar 与 APP 共享 CPU Mem；
- 运维周边建设：
  - 原地注入；
  - 平滑升级；
  - Sidecar 重启；
- 监控建设：
  - 系统指标：CPU、Mem、TCP、Disk IO；
  - Go 指标：Processor、Goroutines、Memstats、GC；
  - RPC 指标：QPS、RT、连接数；
- 旁路增强：
  - 服务注册中心性能提升；

## 回顾

1. 如何让 Service Mesh 发挥最大的业务价值？**保证效率增加成本不变**
1. 如何达成几十万容器接入 SOFAMosn 的目标？**降低接入成本**
1. 如何处理几十万容器 SOFAMosn 的版本升级问题？**降低应用感知**
1. 如何保障 Service Mesh 的性能与稳定性达标？**性能与稳定性层层优化**

通过对以上问题的深入解读，大家可以了解到蚂蚁金服的 Service Mesh 为何可以稳定支撑双十一，也期望能为大家落地 Service Mesh 带来一些不一样的思考。更多 Service Mesh 相关内容敬请关注我们的公众号“金融级分布式架构”来了解更多。
