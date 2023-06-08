# Kubernetes：加速 Pod 启动的方法

## 并行拉取镜像

默认情况下，镜像拉取是串行的。换句话说，kubelet 一次只会向镜像服务发送一个镜像拉取请求，其他的拉取请求必须等到正在处理的请求完成。

kubelet 配置 `serializeImagePulls` 为 `false` 时，可以同时拉取多个镜像。

## 配置并行拉取镜像阈值防止节点过载

要限制同时拉取图像的次数，可以在 `kubelet` 中配置 `maxParallelImagePulls` 字段。通过将 `maxParallelImagePulls` 设置为 `n` 值，将只同时拉取 `n` 个镜像。超出此限制的任何其他镜像拉取都将等待，直到至少一个正在进行的拉取完成。

## 提高 kubelet api qps

为了改进节点上有多个 Pod 的场景下的 Pod 启动，尤其是突然伸缩的情况，Kubelet 需要同步 Pod 状态并准备 configmaps、secrets 或 volumes。这需要很大的带宽来访问 kube-apiserver。

在 v1.27 之前的版本中，默认值 kubeAPIQPS 为 5， kubeAPIBurst 为 10。但是，v1.27 中的 kubelet 将这些默认值分别增加到 50 和 100，以便在 pod 启动期间获得更好的性能。值得注意的是，这并不是我们提高 Kubelet API QPS 限制的唯一原因。

之前，在 pod 启动期间，我们经常在超过 50 个 pod 的节点中遇到 `volume mount timeout` 。我们建议集群操作员将 kubeAPIQPS 调整到 20，kubeAPIBurst 调整到 40。

## 事件触发的容器状态更新

Evented PLEG （PLEG 是“Pod Lifecycle Event Generator”的缩写，Kubernetes 为 kubelet 提供了两种检测 Pod 生命周期事件的方法，例如容器中的最后一个进程关闭。在 Kubernetes v1.27 中，基于事件的机制已升级为 beta，但默认情况下仍处于禁用状态。如果切换到基于事件的生命周期更改检测，则 kubelet 能够比依赖轮询的默认方法更快地启动 Pod。轮询生命周期更改的默认机制增加了明显的开销;这会影响 Kubelet 并行处理不同任务的能力，并导致性能和可靠性问题不佳。

## 提高 Pod 资源限制

在启动期间，某些 Pod 可能会消耗大量 CPU 或内存。如果 CPU 限制较低，会显著减慢 Pod 启动过程。为了改进内存管理，Kubernetes v1.22 为 kubelet 引入了一个名为 MemoryQoS 的功能门。此功能使 kubelet 能够在容器、pod 和 QoS 级别设置内存 QoS，以便在使用 cgroups v2 运行时提供更好的保护和保证内存质量。尽管它有好处，但如果 Pod 启动消耗大量内存，启用此功能门可能会影响 Pod 的启动速度。

kubelet 配置现在包括 `memoryThrottlingFactor` 。此系数乘以内存限制或节点可分配内存，以设置 cgroup v2 memory.high 值以强制实施 MemoryQoS。降低此系数会降低容器 cgroup 的上限，从而增加回收压力。增加此系数将减少回收压力。默认值最初为 0.8，在 Kubernetes v1.27 中将更改为 0.9。此参数调整可以减少此功能对 Pod 启动速度的潜在影响。

## 相关文档

- https://kubernetes.io/blog/2023/05/15/speed-up-pod-startup/