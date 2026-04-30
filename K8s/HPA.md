# HPA

HPA（Horizontal Pod Autoscaler）是 Kubernetes 中的**水平自动扩缩容**机制，根据负载自动增减 Pod 的**数量**。

## 和 VPA 的核心区别

|          | HPA                        | VPA                   |
| -------- | -------------------------- | --------------------- |
| 扩展方式 | 增减 Pod **数量**          | 调整 Pod **资源大小** |
| 适合场景 | 无状态、可横向扩展的服务   | 资源难以估算的应用    |
| 响应延迟 | 较快（秒级）               | 较慢（需驱逐重建）    |
| 典型触发 | CPU/内存使用率、自定义指标 | 历史资源用量分析      |

## 工作原理

HPA 持续监控 metrics，当负载上升时增加 Pod，负载下降时缩减 Pod，始终保持目标指标（比如 CPU 使用率 70%）在阈值附近。

## 基本用法

**方式一：命令行快速创建**

```bash
kubectl autoscale deployment my-app \
  --min=2 --max=10 --cpu-percent=70
```

**方式二：YAML 声明式配置（推荐）**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70   # CPU 使用率超过 70% 就扩容
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**查看状态：**

```bash
kubectl get hpa
kubectl describe hpa my-app-hpa
```

## 前提条件

HPA 要正常工作，Pod 必须设置 `resources.requests`，否则无法计算使用率百分比——这也是 VPA 建议值的用武之地：先用 VPA 的 `Off` 模式跑一段时间，拿到合理的 requests 基准值，再交给 HPA 做动态扩缩容，两者配合效果最好。