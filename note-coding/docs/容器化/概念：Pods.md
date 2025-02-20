# 名词解释 Pods

> 来源：https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#working-with-pods

在Kubernetes中，最小的管理元素不是一个个独立的容器，而是Pod,Pod是最小的，管理，创建，计划的最小单元。

**Pod**（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） [容器](https://kubernetes.io/zh-cn/docs/concepts/overview/what-is-kubernetes/#why-containers)； 这些容器共享存储、网络、以及怎样运行这些容器的声明。

Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的 “逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 [Init 容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/)。 你也可以在集群支持[临时性容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/ephemeral-containers/)的情况下， 为调试的目的注入临时性容器。



## 什么是 Pod？

Pod的共享上下文包括一组Linux命名空间、控制组（cgoup）和可能一些其他的隔离方面，即用来隔离容器的技术。在Pod的上下文中，每个独立的应用可能会进一步实时隔离。

Pod 类似于共享名字空间并共享文件系统卷的一组容器。

## 使用Pod

下面是一个 Pod 示例，它由一个运行镜像 `nginx:1.14.2` 的容器组成。

[`pods/simple-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/pods/simple-pod.yaml) 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```

要创建上面显示的 Pod，请运行以下命令：

```
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

Pod 通常不是直接创建的，而是使用工作负载资源创建的。 有关如何将 Pod 用于工作负载资源的更多信息，请参阅[使用 Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#working-with-pods)。

### 用于管理 pod 的工作负载资源

通常你不需要直接创建 Pod，甚至单实例 Pod。 相反，你会使用诸如 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 或 [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 这类工作负载资源来创建 Pod。 如果 Pod 需要跟踪状态，可以考虑 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/) 资源。

Kubernetes 集群中的 Pod 主要有两种用法：

- 运行单个容器的Pod。 "每个 Pod 一个容器" 模型是最常见的 Kubernetes 用例； 在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。

- 运行多个协同工作的容器的 Pod。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众， 而另一个单独的 “边车”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。

> **说明：**
>
> 将多个并置、同管的容器组织到一个 Pod 中是一种相对高级的使用场景。 只有在一些场景中，容器之间紧密关联时你才应该使用这种模式。

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序 （例如，运行多个实例以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。 在 Kubernetes 中，这通常被称为**副本（Replication）**。 通常使用一种工作负载资源及其[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)来创建和管理一组 Pod 副本。

参见 [Pod 和控制器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pods-and-controllers)以了解 Kubernetes 如何使用工作负载资源及其控制器以实现应用的扩缩和自动修复。

### Pod 怎样管理多个容器

Pod 被设计成支持形成内聚服务单元的多个协作过程（形式为容器）。 Pod 中的容器被自动安排到集群中的同一物理机或虚拟机上，并可以一起进行调度。 容器之间可以共享资源和依赖、彼此通信、协调何时以及何种方式终止自身。

例如，你可能有一个容器，为共享卷中的文件提供 Web 服务器支持，以及一个单独的 "边车 (sidercar)" 容器负责从远端更新这些文件，如下图所示：

![Pod 创建示意图](.\imgs\pod.svg)

有些 Pod 具有 [Init 容器](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-init-container)和 [应用容器](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-app-container)。 Init 容器会在启动应用容器之前运行并完成。

Pod 天生地为其成员容器提供了两种共享资源：[网络](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-networking)和[存储](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-storage)。

## 使用 Pod

你很少在 Kubernetes 中直接创建一个个的 Pod，甚至是单实例（Singleton）的 Pod。 这是因为 Pod 被设计成了相对临时性的、用后即抛的一次性实体。 当 Pod 由你或者间接地由[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 创建时，它被调度在集群中的[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上运行。 Pod 会保持在该节点上运行，直到 Pod 结束执行、Pod 对象被删除、Pod 因资源不足而被**驱逐**或者节点失效为止。

```
说明： 重启 Pod 中的容器不应与重启 Pod 混淆。 Pod 不是进程，而是容器运行的环境。 在被删除之前，Pod 会一直存在。
```

Pod 的名称必须是一个合法的 [DNS 子域](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)值， 但这可能对 Pod 的主机名产生意外的结果。为获得最佳兼容性，名称应遵循更严格的 [DNS 标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-label-names)规则。

### Pod 操作系统

**特性状态：** `Kubernetes v1.25 [stable]`

你应该将 `.spec.os.name` 字段设置为 `windows` 或 `linux` 以表示你希望 Pod 运行在哪个操作系统之上。 这两个是 Kubernetes 目前支持的操作系统。将来，这个列表可能会被扩充。

在 Kubernetes v1.27 中，为此字段设置的值对 Pod 的[调度](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-scheduler/)没有影响。 设置 `. spec.os.name` 有助于确定性地标识 Pod 的操作系统并用于验证。 如果你指定的 Pod 操作系统与运行 kubelet 所在节点的操作系统不同， 那么 kubelet 将会拒绝运行该 Pod。 [Pod 安全标准](https://kubernetes.io/zh-cn/docs/concepts/security/pod-security-standards/)也使用这个字段来避免强制执行与该操作系统无关的策略。

### Pod 和控制器

你可以使用工作负载资源来创建和管理多个 Pod。 资源的控制器能够处理副本的管理、上线，并在 Pod 失效时提供自愈能力。 例如，如果一个节点失败，控制器注意到该节点上的 Pod 已经停止工作， 就可以创建替换性的 Pod。调度器会将替身 Pod 调度到一个健康的节点执行。

下面是一些管理一个或者多个 Pod 的工作负载资源的示例：

- [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/)

### Pod 模板

[工作负载](https://kubernetes.io/zh-cn/docs/concepts/workloads/)资源的控制器通常使用 **Pod 模板（Pod Template）** 来替你创建 Pod 并管理它们。

Pod 模板是包含在工作负载对象中的规范，用来创建 Pod。这类负载资源包括 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)、 [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 和 [DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/) 等。

工作负载的控制器会使用负载对象中的 `PodTemplate` 来生成实际的 Pod。 `PodTemplate` 是你用来运行应用时指定的负载资源的目标状态的一部分。

下面的示例是一个简单的 Job 的清单，其中的 `template` 指示启动一个容器。 该 Pod 中的容器会打印一条消息之后暂停。

```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # 这里是 Pod 模板
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # 以上为 Pod 模板
```

修改 Pod 模板或者切换到新的 Pod 模板都不会对已经存在的 Pod 直接起作用。 如果改变工作负载资源的 Pod 模板，工作负载资源需要使用更新后的模板来创建 Pod， 并使用新创建的 Pod 替换旧的 Pod。

例如，StatefulSet 控制器针对每个 StatefulSet 对象确保运行中的 Pod 与当前的 Pod 模板匹配。如果编辑 StatefulSet 以更改其 Pod 模板， StatefulSet 将开始基于更新后的模板创建新的 Pod。

每个工作负载资源都实现了自己的规则，用来处理对 Pod 模板的更新。 如果你想了解更多关于 StatefulSet 的具体信息， 请阅读 StatefulSet 基础教程中的[更新策略](https://kubernetes.io/zh-cn/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets)。

在节点上，[kubelet](https://kubernetes.io/docs/reference/generated/kubelet) 并不直接监测或管理与 Pod 模板相关的细节或模板的更新，这些细节都被抽象出来。 这种抽象和关注点分离简化了整个系统的语义， 并且使得用户可以在不改变现有代码的前提下就能扩展集群的行为。

## Pod 更新与替换

正如前面章节所述，当某工作负载的 Pod 模板被改变时， 控制器会基于更新的模板创建新的 Pod 对象而不是对现有 Pod 执行更新或者修补操作。

Kubernetes 并不禁止你直接管理 Pod。对运行中的 Pod 的某些字段执行就地更新操作还是可能的。不过，类似 [`patch`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#patch-pod-v1-core) 和 [`replace`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#replace-pod-v1-core) 这类更新操作有一些限制：

- Pod 的绝大多数元数据都是不可变的。例如，你不可以改变其 `namespace`、`name`、 `uid` 或者 `creationTimestamp` 字段；`generation` 字段是比较特别的， 如果更新该字段，只能增加字段取值而不能减少。
- 如果 `metadata.deletionTimestamp` 已经被设置，则不可以向 `metadata.finalizers` 列表中添加新的条目。
- Pod 更新不可以改变除 `spec.containers[*].image`、`spec.initContainers[*].image`、 `spec.activeDeadlineSeconds` 或 `spec.tolerations` 之外的字段。 对于 `spec.tolerations`，你只被允许添加新的条目到其中。
- 在更新 `spec.activeDeadlineSeconds` 字段时，以下两种更新操作是被允许的：
  1. 如果该字段尚未设置，可以将其设置为一个正数；
  2. 如果该字段已经设置为一个正数，可以将其设置为一个更小的、非负的整数。

### 资源共享和通信

Pod 使它的成员容器间能够进行数据共享和通信。

### Pod 中的存储

一个 Pod 可以设置一组共享的存储[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)。 Pod 中的所有容器都可以访问该共享卷，从而允许这些容器共享数据。 卷还允许 Pod 中的持久数据保留下来，即使其中的容器需要重新启动。 有关 Kubernetes 如何在 Pod 中实现共享存储并将其提供给 Pod 的更多信息， 请参考[存储](https://kubernetes.io/zh-cn/docs/concepts/storage/)。

### Pod 联网

每个 Pod 都在每个地址族中获得一个唯一的 IP 地址。 Pod 中的每个容器共享网络名字空间，包括 IP 地址和网络端口。 **Pod 内**的容器可以使用 `localhost` 互相通信。 当 Pod 中的容器与 **Pod 之外**的实体通信时，它们必须协调如何使用共享的网络资源（例如端口）。

在同一个 Pod 内，所有容器共享一个 IP 地址和端口空间，并且可以通过 `localhost` 发现对方。 他们也能通过如 SystemV 信号量或 POSIX 共享内存这类标准的进程间通信方式互相通信。 不同 Pod 中的容器的 IP 地址互不相同，如果没有特殊配置，就无法通过 OS 级 IPC 进行通信。 如果某容器希望与运行于其他 Pod 中的容器通信，可以通过 IP 联网的方式实现。

Pod 中的容器所看到的系统主机名与为 Pod 配置的 `name` 属性值相同。 [网络](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/networking/)部分提供了更多有关此内容的信息。

## 容器的特权模式

**说明：**

你的[容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes)必须支持特权容器的概念才能使用这一配置。

Pod 中的所有容器都可以在特权模式下运行，以使用原本无法访问的操作系统管理权能。 此模式同时适用于 Windows 和 Linux。

### Linux 特权容器

在 Linux 中，Pod 中的所有容器都可以使用容器规约中的 [安全性上下文](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/)中的 `privileged`（Linux）参数启用特权模式。 这对于想要使用操作系统管理权能（Capabilities，如操纵网络堆栈和访问硬件设备）的容器很有用。

### Windows 特权容器

**特性状态：** `Kubernetes v1.26 [stable]`

在 Windows 中，你可以使用 Pod 规约中安全上下文的 `windowsOptions.hostProcess` 参数来创建 [Windows HostProcess Pod](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/create-hostprocess-pod/)。 这些 Pod 中的所有容器都必须以 Windows HostProcess 容器方式运行。 HostProcess Pod 可以直接运行在主机上，它也能像 Linux 特权容器一样，用于执行管理任务。

## 静态 Pod

**静态 Pod（Static Pod）** 直接由特定节点上的 `kubelet` 守护进程管理， 不需要 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)看到它们。 尽管大多数 Pod 都是通过控制面（例如，[Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)） 来管理的，对于静态 Pod 而言，`kubelet` 直接监控每个 Pod，并在其失效时重启之。

静态 Pod 通常绑定到某个节点上的 [kubelet](https://kubernetes.io/docs/reference/generated/kubelet)。 其主要用途是运行自托管的控制面。 在自托管场景中，使用 `kubelet` 来管理各个独立的[控制面组件](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#control-plane-components)。

`kubelet` 自动尝试为每个静态 Pod 在 Kubernetes API 服务器上创建一个[镜像 Pod](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-mirror-pod)。 这意味着在节点上运行的 Pod 在 API 服务器上是可见的，但不可以通过 API 服务器来控制。

**说明：**

静态 Pod 的 `spec` 不能引用其他的 API 对象（例如： [ServiceAccount](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-service-account/)、 [ConfigMap](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/)、 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/) 等）。

## 容器探针

**Probe** 是由 kubelet 对容器执行的定期诊断。要执行诊断，kubelet 可以执行三种动作：

- `ExecAction`（借助容器运行时执行）
- `TCPSocketAction`（由 kubelet 直接检测）
- `HTTPGetAction`（由 kubelet 直接检测）

你可以参阅 Pod 的生命周期文档中的[探针](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)部分。

## 接下来

- 了解 [Pod 生命周期](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/)。
- 了解 [RuntimeClass](https://kubernetes.io/zh-cn/docs/concepts/containers/runtime-class/)， 以及如何使用它来配置不同的 Pod 使用不同的容器运行时配置。
- 了解 [PodDisruptionBudget](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/disruptions/)， 以及你可以如何利用它在出现干扰因素时管理应用的可用性。
- Pod 在 Kubernetes REST API 中是一个顶层资源。 [Pod](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/pod-v1/) 对象的定义中包含了更多的细节信息。
- 博客[分布式系统工具箱：复合容器模式](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)中解释了在同一 Pod 中包含多个容器时的几种常见布局。
- 了解 [Pod 拓扑分布约束](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/topology-spread-constraints/)。

要了解为什么 Kubernetes 会在其他资源 （如 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/) 或 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)） 封装通用的 Pod API，相关的背景信息可以在前人的研究中找到。具体包括：

- [Aurora](https://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)
- [Borg](https://research.google.com/pubs/pub43438.html)
- [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html)
- [Omega](https://research.google/pubs/pub41684/)
- [Tupperware](https://engineering.fb.com/data-center-engineering/tupperware/).