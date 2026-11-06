# Cluster Autoscaler：调试与性能

## 实时可见性日志

- **资源：** `assets/log-autoscaler-events.sh <cluster-name>`（实时跟踪）。

## `messageId` 速查表
| ID | 含义 | 修复方法 |
|----|---------|-----|
| `scale.up.error.out.of.resources` | GCE 库存不足 | 在 ComputeClass 中添加可用区/机型系列回退。 |
| `scale.up.error.quota.exceeded` | 项目配额上限 | 提高区域配额。 |
| `scale.up.error.ip.space.exhausted` | 子网已满 | 扩展 Pod IP 范围。 |
| `scale.up.no.scale.up` | 无优先级匹配 | 检查 Pod 请求与 ComputeClass 边界。 |

## Pending Pod 检查清单
1. `kubectl describe pod`：检查事件中是否有 "insufficient cpu" 或 "taints"。
2. **触达 `--max-nodes`？** 检查节点池限制。
3. **选择器冲突？** Pod 固定 `gke-spot=true`，而 ComputeClass 为 On-Demand。
4. **节点池自动创建已启用？** 检查 `nodePoolAutoCreation.enabled: true`。
5. **可见性日志：** 读取 `noDecisionStatus.noScaleUp` 获取确切的拒绝原因。
6. **EKS 到 GKE 选择器转换：** 若从 EKS/Karpenter 迁移，确保将 AWS 风格或通用选择器（`machine-family`）转换为 GKE 原生选择器（`cloud.google.com/machine-family`）。`scale.up.no.scale.up` 的常见原因是 Pod 请求 `machine-family: c3`，而 GKE 只识别 `cloud.google.com/machine-family: c3`。
7. **机器系列支持：** 若节点池自动创建无法为特定 `machineFamily` 或 `instance-type`（如 N4、C3A）提供节点，请确认 GKE 版本支持该系列用于节点池自动创建 / Autopilot。旧版 GKE 会忽略不支持的系列。查阅 GKE 发布说明或节点池自动创建文档了解版本要求。
8. **全新预留？** 最近约 30 分钟内创建的预留可能尚未进入 Cluster Autoscaler 缓存。在缓存追上之前针对它会导致 Cluster Autoscaler 放弃该预留并停滞。创建预留后等待 **≥30 分钟** 再驱动扩容（见 `ca-optimization.md`）。

## 查找缩容阻塞因素

- **资源：** `./assets/find-scale-down-blockers.sh`（扫描集群中的阻塞因素）。

### 常见原因
- **裸 Pod：** 无控制器（Deployment/Job）；autoscaler 不会驱逐。
- **本地存储：** 本地 SSD 上的 `emptyDir` 或 `hostPath`。
- **注解：** `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"`。
- **PDB：** 当前允许零中断。
- **下限：** `min-nodes` 或 `total-min-nodes` > 0。

## 性能与迟缓
- **强制反亲和性：** 大规模下调度成本激增。使用 `preferred` 或 `topologySpreadConstraints`。
- **节点池数量：** 超过约 200 个节点池后，自动扩缩容会变慢。合并近乎重复的 ComputeClass。
- **Spot 宽限期：** 抢占通知约 30 秒，无法通过 ComputeClass 字段延长。将 `terminationGracePeriodSeconds` 和 SIGTERM 处理保持在此范围内；依靠副本和 PDB 吸收波动。

## 隔离系统 Pod（专家模式）
症状：`kube-system` Pod（metrics-server、coredns）落在昂贵节点上并将其固定。
**修复：** 通过命名空间默认 ComputeClass 隔离。
1. 应用一个"廉价"的 `system-pool` ComputeClass。
2. 为 `kube-system` 命名空间打标签：
   `kubectl label ns kube-system cloud.google.com/default-compute-class-non-daemonset=system-pool`