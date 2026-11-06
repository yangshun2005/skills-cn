# Cluster Autoscaler：优化配置与位置策略

## 自动扩缩配置（集群级）
| 配置 | 行为 | 适用场景 |
|---------|----------|-------------|
| `balanced`（默认） | 保留备用容量；缩容保守。 | 延迟敏感型服务。 |
| `optimize-utilization` | 激进装箱；更快移除节点。 | 成本驱动；批处理；**Golden Path**。 |

- **命令：** `gcloud container clusters update <C> --autoscaling-profile=optimize-utilization`。

## Spot 最佳实践
- **强制回退：** 任何时候推荐或调试 Spot 实例的使用（无论是通过手动节点池还是节点池自动创建），都应建议通过 ComputeClass 加入其他 Spot 或按需回退优先级。Spot 容量波动很大，没有回退时，工作负载会在 GCE 库存不足（`scale.up.error.out.of.resources`）期间卡住。

## 位置策略（`--location-policy`）

- **`BALANCED`**：在*扩容时*尽力实现跨可用区的**节点**均匀分布（基础设施层；某可用区容量不足时仍会扩容）。**不**均衡 **Pod**——那是独立的工作负载层，需要 Pod 的 `topologySpreadConstraints`（`DoNotSchedule`，见下文 PTS）。两者相互独立。用于 **HA 工作负载**。
- **`ANY`**：从任意可用区获取容量。**最适合 Spot VM** 和稀缺 SKU（最大化可获取性）。

## ComputeClass `locationPolicy`

```yaml
priorities:
- machineFamily: n4
  spot: true
  location:
    locationPolicy: ANY # Spot preference
```

## Pod 拓扑分布约束（PTS）
Cluster Autoscaler 支持 PTS，用于扩容时的可用区分布。

要通过自动扩缩器强制分布，你**必须**使用 `whenUnsatisfiable: DoNotSchedule`。

示例配置：
```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: "topology.kubernetes.io/zone"
    whenUnsatisfiable: DoNotSchedule  # Required for cluster autoscaler compatibility
    labelSelector:
      matchLabels:
        app: my-app
```

## 资源 CUD 与预留

- **承诺使用折扣（CUD）：** 由 Cluster Autoscaler 自动消耗。当自动扩缩器为特定机型系列（如 `n4`）创建节点时，会自动消耗该系列所有可用 CUD，直至耗尽。无需显式配置自动扩缩器、节点自动预配或 ComputeClass。
- **预留：** 与 CUD 不同，容量预留**不会**被自动消耗。必须显式指定。你必须通过 Node Pool API（用于标准/手动节点池）或 ComputeClass 的 `reservations` 块（用于节点池自动创建）配置消耗。
- **新建预留（缓存延迟）：** 自动扩缩器会缓存预留数据，**不会**立即看到新预留。在 Cluster Autoscaler 缓存过期时针对全新预留驱动扩容，会导致 Cluster Autoscaler 找不到容量并**对该预留退避**——这会延迟重试并加剧停滞。**修复：** 创建预留后**至少等待 30 分钟**，再依赖它进行自动扩缩器驱动的扩容。（适用于预留本身；扩展现有已缓存预留没有问题。）