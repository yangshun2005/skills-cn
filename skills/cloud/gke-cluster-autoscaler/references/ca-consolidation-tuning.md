# Cluster Autoscaler：整合调优

## `autoscalingPolicy`（ComputeClass）
覆盖由 ComputeClass 管理的节点的集群级配置默认值。
```yaml
spec:
  autoscalingPolicy:
    consolidationDelayMinutes: 5     # 下限 = 1 分钟
    consolidationThreshold: 0        # CPU 利用率百分比（0 = 始终作为候选）
    gpuConsolidationThreshold: 0     # 加速器对应项
```

## 按工作负载调优
- **在线服务：** 延迟 5–15 分钟；使用默认阈值。防止流量高峰时反复抖动。
- **批处理：** 延迟 1–2 分钟；阈值设为 `0`。积极回收成本。
- **有状态服务：** 延迟 10 分钟以上。配合 PDB 控制节点变动。

## 中断约束
整合会遵循：
- **PodDisruptionBudgets（PDB）：** 若驱逐会违反 `maxUnavailable`，则跳过该节点。
- **`safe-to-evict: "false"`：** 该注解会无限期固定节点。

*注意：* 维护窗口**不会**阻止整合。如需按时间窗口抑制，请使用 PDB。