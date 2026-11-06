# Cluster Autoscaler：节点配置与策略

## 启用扩缩容（Standard）

### cluster autoscaler - 按节点池

- **启用（新节点池）：**
  ```bash
  gcloud container node-pools create <POOL> \
    --enable-autoscaling --min-nodes=1 --max-nodes=10
  ```
- **启用（现有节点池）：**
  ```bash
  gcloud container clusters update <CLUSTER> \
    --enable-autoscaling --node-pool=<POOL> \
    --min-nodes=1 --max-nodes=10
  ```

### 节点自动配置 - 集群级

- **启用：**
  ```bash
  gcloud container clusters update <CLUSTER> \
    --enable-autoprovisioning \
    --min-cpu=4 --max-cpu=200 \
    --min-memory=16 --max-memory=800
  ```

### 节点池自动创建 - 按 ComputeClass

- **启用：** 在 ComputeClass 中设置 `nodePoolAutoCreation.enabled: true`。
- **GKE 1.33.3+：** 无需启用集群级节点自动配置即可生效。

## 节点配置策略

| 策略 | 优势 | 适用场景 |
|----------|-----------|----------|
| **手动节点池** | 调度快；名称稳定。 | 延迟敏感；手动管理。 |
| **节点池自动创建（ComputeClass）** | 可获取性最佳；可缩容至零。 | 突发负载；批处理；成本敏感。 |
| **混合** | 手动节点池优先；节点池自动创建兜底。 | **推荐用于生产环境。** |

## 切换：从节点自动配置迁移到节点池自动创建
1. **应用 ComputeClass：** 创建类并设置 `nodePoolAutoCreation.enabled: true`。
2. **工作负载启用：** 应用 `nodeSelector: cloud.google.com/compute-class: <name>`。
3. **排空旧节点池：** 对旧节点自动配置管理的节点池中的节点执行 `kubectl drain`。

## 缩容至零行为
- **手动节点池：** 标准 cluster autoscaler 会保留至少 1 个节点，除非支持并启用了空节点池删除。
- **节点池自动创建管理：** 节点池为空时，Autoscaler 可删除整个节点池。