# Cluster Autoscaler：容量缓冲（预热）

**发布阶段：预览版**（Pre-GA Offerings Terms）。版本门槛：活跃缓冲需要 GKE **1.35.2-gke.1842000+**；待机缓冲需要 GKE **1.36.0-gke.2253000+**。

## `CapacityBuffer`（CRD）

- **API：** `autoscaling.x-k8s.io/v1beta1`，`kind: CapacityBuffer`。**命名空间级。**
- 通过 `podTemplateRef` 中的 `nodeSelector` 定位特定 `ComputeClass`；可与节点池自动创建配合使用（缓冲遵循该类的 `priorities[]`）。
- **两种 `provisioningStrategy` 取值**（不要混淆这两个域）：
  - **活跃：** `buffer.x-k8s.io/active-capacity`（默认）——占位 Pod 持有完全**运行中**的热节点；真实工作负载会立即驱逐它们。空闲时按完整节点价格付费。
  - **待机：** `buffer.gke.io/standby-capacity`（2026 年 6 月）——节点完全初始化（DaemonSet 已启动、镜像已拉取），然后**挂起**。

## 待机缓冲（`buffer.gke.io/standby-capacity`）

- **成本模型：** 挂起期间只需支付**持久磁盘 + IP 地址**（节点状态存于磁盘；计算/内存费用停止）。恢复耗时**约 30 秒**——比新建节点快 2-3 倍。
- **要求：** GKE **1.36.0-gke.2253000+**；启用节点自动预配/节点池自动创建的 **Standard** 集群。ComputeClass 定位方式与活跃缓冲相同。
- **注解（作用于 CapacityBuffer）：**
  - `buffer.gke.io/standby-capacity-init-time`（默认 `"5m"`）：新建（或重建）的缓冲节点在挂起前保持活跃的时长。
  - `buffer.gke.io/standby-capacity-refresh-frequency`（默认 `"24h"`；`"never"` 表示禁用）：待机节点的重建频率。
- **检测：** 挂起的节点带有节点状况 **`Suspended`**（`True` = 已挂起，`False` = 已恢复，缺失 = 从未挂起）。
- **混合模式：** 小活跃缓冲 + 大待机缓冲——GKE 通过恢复待机节点（约 30 秒）补充活跃缓冲，并在后台回填待机缓冲。
- **排除项：** 不支持 GPU/TPU 节点、Local SSD、CMEK 或 Confidential GKE Nodes；适用 Compute Engine 挂起/恢复限制（例如内存 >208 GB 的节点）。加速器预热必须使用**活跃**缓冲。

## 规模模式

- **固定：** `replicas: 3`。始终保持 N 个单元预热。
- **动态：** `percentage: 20` + `scalableRef: <Deployment>`。预留容量随工作负载伸缩。
- **资源限制：** `limits: {cpu, memory}` 限制缓冲的总容量。

## 为什么使用缓冲？

- **突发型服务：** Pod 待调度 SLO 无法容忍 60-120 秒的节点池自动创建延迟。
- **HPA 快于集群自动扩缩器：** 工作负载的伸缩速度快于节点到位速度。
- **预热：** 在已知流量窗口前预热 GPU/TPU（仅限活跃缓冲）。
- **待机 = 低成本预留容量：** 若工作负载能容忍约 30 秒的恢复延迟，可以磁盘+IP 成本而非完整节点成本获得超额预配的热容量。

*注意：* 用形状感知、面向类的热容量取代 `--min-nodes` 的"笨"下限。

*相关：* 从 GKE **1.34.1-gke.1829001** 起，节点池自动创建本身最高**快 85%**（并发创建多个节点池；无需配置）。